### 📊 SAP B1 Core Tables to Dimensional Star Schema Mapping
The diagram below illustrates the Entity Relationship Diagram (ERD) of the source SAP B1 tables and how they are transformed into a clean, high-performance Star Schema for executive analytics.

---

## 📊 Data Architecture & Dimensional Modeling (ERD)

To achieve a consolidated view of multi-branch operations, the data architecture transitions from an **OLTP (Transactional) environment** native to SAP Business One into an **OLAP (Analytical) Star Schema** housed within the centralized staging database.

The Entity Relationship Diagram (ERD) below outlines the structural mapping and logical pipeline:

```mermaid
erDiagram
    %% --- LAPISAN SUMBER: DATA MASTER SAP B1 ---
    OCRD_Customer_Master {
        string CardCode PK "Kode Pelanggan"
        string CardName "Nama Pelanggan"
        string GroupCode "Grup Wilayah"
    }
    OITM_Item_Master {
        string ItemCode PK "Kode Barang"
        string ItemName "Nama Barang"
        string ItmsGrpCod "Kategori Produk"
    }

    %% --- LAPISAN SUMBER: DATA TRANSAKSI SAP B1 ---
    OINV_Invoice_Header {
        int DocEntry PK "ID Internal"
        int DocNum "Nomor Invoice"
        date DocDate "Tanggal Transaksi"
        string CardCode FK "Link ke OCRD"
        double DocTotal "Total Penjualan"
    }
    INV1_Invoice_Rows {
        int DocEntry PK, FK "Link ke OINV"
        int LineNum PK "Nomor Baris"
        string ItemCode FK "Link ke OITM"
        double Price "Harga Satuan"
        double Quantity "Jumlah Barang"
        double LineTotal "Total per Baris"
    }

    %% --- HUBUNGAN TABEL ASLI SAP B1 ---
    OCRD_Customer_Master ||--o{ OINV_Invoice_Header : "places"
    OINV_Invoice_Header ||--|{ INV1_Invoice_Rows : "contains"
    OITM_Item_Master ||--o{ INV1_Invoice_Rows : "ordered_in"


    %% --- LAPISAN TUJUAN: STAR SCHEMA (STAGING DATABASE) ---
    Fact_Sales {
        int SalesID PK "Auto Increment"
        date DateKey FK "Link ke Dim_Time"
        string CustomerKey FK "Link ke Dim_Customer"
        string ProductKey FK "Link ke Dim_Product"
        string BranchKey FK "Link ke Dim_Branch"
        double NetRevenue "Revenue Bersih"
        int QtySold "Total Kuantitas"
    }

    %% --- PROSES ETL TRANSFORMS ---
    INV1_Invoice_Rows ||..o{ Fact_Sales : "ETL Consolidates & Transforms into"
```
### 🧠 Architectural Breakdown & Logic

1. **Source Layer (Standard SAP B1 Schema):** The pipeline ingests operational transactional data straight from standard SAP B1 relational tables (`OINV`, `INV1`) along with master data definitions (`OCRD`, `OITM`) across all branch databases. 
2. **ETL Consolidation:** Custom ETL scripts execute routine schedules to extract, deduplicate, and merge separate multi-branch databases into a unified central staging layer.
3. **Dimensional Layer (Star Schema):** Complex transactional dependencies are flattened into a high-performance Star Schema. The `Fact_Sales` table isolates numerical metrics (Net Revenue, Quantities), while foreign keys link directly to independent dimensions (`Dim_Customer`, `Dim_Product`, `Dim_Branch`, `Dim_Time`). This structured optimization allows executive end-users to perform seamless, instantaneous multi-attribute pivot analytics without straining production ERP resources.

---

### ⚠️ Data Privacy & Legal Compliance Disclaimer
*To ensure absolute compliance with corporate Non-Disclosure Agreements (NDA) and strict data protection standards, this portfolio adheres to the following privacy rules:*
* All tables and structural fields are limited to **globally standardized SAP Business One core schemas**, omitting any company-specific custom tables or internal configurations.
* No real production data, actual financial margins, or real client names are exposed.
* All data points hosted within the `/data` folder are entirely **anonymized, synthetically generated, and randomized** solely for data engineering simulation and pipeline demonstration purposes.

---