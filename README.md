This is the TIL project for TIL6022 at the TU Delft

**Contributors**:


[@yme116](https://github.com/yme116)

[@bartligtenberg](https://github.com/bartligtenberg)

[@HuubKav](https://github.com/HuubKav)

[@Gijsjjmeijers](https://github.com/Gijsjjmeijers)

**Layout**:

[Coding finished](https://github.com/HuubKav/TIL6022_python_programming/tree/main/Coding%20finished): This is the map where all codes, and files needed to run the codes are stored.

[Figures](https://github.com/HuubKav/TIL6022_python_programming/tree/main/Figures): This is the map where all figures are stored.

[Assignment paper.ipynb](https://github.com/HuubKav/TIL6022_python_programming/blob/main/Assignment%20paper.ipynb): This is the final report jupiter file.

[jeroen_punt_nl_dynamische_stroomprijzen_jaar_2024.csv](https://github.com/HuubKav/TIL6022_python_programming/blob/main/jeroen_punt_nl_dynamische_stroomprijzen_jaar_2024.csv): This is the dataset used for this project.

# Introduction

Unmanaged home charging of electric vehicles typically follows a plug-and-charge pattern: drivers connect the vehicle upon arrival, and charging starts immediately until the battery is full.  
While convenient, this approach ignores substantial intra-day variation in electricity prices. Aligning charging decisions with hourly price signals can reduce household expenditures without compromising vehicle availability.

**Main research question**

> How much money can an individual EV owner save yearly by optimizing their charging schedule based on the hourly price of electricity?

To answer this question, we:
1. Explore how electricity prices fluctuate within a day and across the seasons throughout 2024.  
2. Compare yearly charging costs for different energy contract types — both fixed and dynamic.  
3. Develop an optimization approach that determines the most cost-efficient charging schedule under realistic daily constraints such as arrival and departure time, battery state, and charging power.

To capture a range of real-world usage patterns, we model four representative EV user types:  
**the Distant Commuter, the City Hopper, the Road Tripper, and the Grocery Grabber.**  
These profiles differ in driving frequency and daily distance, which directly affect charging needs and potential cost savings.

The next section describes these user scenarios and outlines the assumptions and constraints used in the analysis.

---

# User Scenarios and Experimental Setup

To evaluate potential cost savings across different driving habits, four representative user scenarios are defined.  
These profiles capture a broad range of real-world driving behaviors and energy demands.  
Each scenario differs in the  distance driven, driving frequency, and consequently, the energy required per charging session.

### General Assumptions

- All users charge at home using a **7.4 kW single-phase wallbox**.  
- Vehicle efficiency: **0.17 kWh per km**.  
- Charging window: **18:00 – 07:00** (arrival to departure).  
- The vehicle must reach the **target state of charge by 07:00** each driving day.  
- Electricity price data: **hourly prices for 2024 from [jeroen.nl](https://www.jeroen.nl/)** (dynamic prices in the Netherlands).  

The four user types are categorized by two behavioral dimensions:  
**frequency of car use** and **distance driven per trip.**
<p align="center">
  <img src="Figures/Types of commuters.png" alt="Figure 1. EV user classification" width="250">
</p>

<p align="center"><b>Figure 1.</b> Illustration of how each EV user type fits within the behavioral spectrum of driving frequency and distance.</p>



---

### Key Characteristics of Each Scenario

| User Type | Description | Driving Pattern | Avg. Distance per Day (km) | Energy per Charge (kWh) | Approx. Charging Time |
|:-----------|:-------------|:----------------|:----------------------------|:------------------------|:----------------------|
| **Distant Commuter** | Daily, long trips | Uses car every day | 174 | 29.6 | 4 h |
| **City Hopper** | Daily, short trips | Uses car every day | 43.5 | 7.4 | 1 h |
| **Road Tripper** | Weekend, long trips | Uses car Sat + Sun | 174 | 29.6 | 4 h |
| **Grocery Grabber** | Twice per week, short trips | Uses car Wed + Sun | 43.5 | 7.4 | 1 h |

---

### Electricity Contract Types

Each user scenario is analyzed under **three contract types**:

1. **Fixed-price contract**  
   - Traditional one-year plan with a constant rate.  

2. **Dynamic-price contract (not optimized)**  
   - Charging begins immediately upon arrival (“plug-and-charge”).  

3. **Dynamic-price contract (optimized)**  
   - Charging schedule is optimized to minimize cost within the allowed charging window.

---

These scenarios and contract types form the foundation for the cost analysis presented in the next section, where **annual electricity expenses** are calculated and compared for all user types.

---


This research uses two types of data to compare fixed and dynamic electricity contracts.

---

### 1. Dynamic Hourly Electricity Prices

The first dataset is obtained from [jeroen.nl](https://jeroen.nl/account/feeds),  
an independent Dutch information and comparison platform focused on energy and electricity.  

Founded in 2023 by energy expert Jeroen Bakker in Groningen, the platform provides insights into:
- Energy behavior and dynamic energy tariffs  
- Home automation and battery use  
- Independent data analysis and consumer tools for sustainable energy decisions  

The dataset used is titled:



It contains dynamic hourly electricity prices (€/kWh) for the Netherlands for the year 2024.  
The key columns are:

| Column | Description |
|:--------|:-------------|
| **datum_nl** | Timestamp in local Dutch time (CET/CEST), indicating the exact date and hour of the price. |
| **datum_utc** | Corresponding timestamp in Coordinated Universal Time (UTC). |
| **prijs_excl_belastingen** | Electricity price in euros per kilowatt-hour (€/kWh), excluding taxes, surcharges, and VAT. |

This dataset forms the basis for the dynamic pricing analysis in this study.

---

### 2. Fixed-Price Electricity Contract

The second source is [energievergelijk.nl](https://www.energievergelijk.nl/nieuws/dit-zijn-de-nieuwe-energietarieven-per-1-december-2023-stroom-en-gas),  
a Dutch comparison website for electricity and gas providers.  

From this source, the cheapest available one-year fixed-price electricity contract as of 1 December 2023 was identified.  
This contract, offered by Vattenfall, specifies a rate of:

> **€0.289 per kWh (including taxes, surcharges, and VAT)**

In this research, this value is used as the fixed electricity rate for 2024.

---

Together, these datasets enable a direct comparison between fixed and dynamic pricing strategies for home EV charging.

---

# Data Pipeline

### Data Preparation and Tax Adjustment

The dataset from [jeroen.nl](https://jeroen.nl/account/feeds) is almost immediately ready for use in this research.  
The only difference between this dataset and the fixed-price data from [energievergelijk.nl](https://www.energievergelijk.nl/nieuws/dit-zijn-de-nieuwe-energietarieven-per-1-december-2023-stroom-en-gas) is that:

- The dynamic prices exclude taxes, surcharges, and VAT.  
- The fixed price includes all of these components.

To ensure a fair comparison, the dynamic prices are adjusted to include taxes, surcharges, and VAT using the following formula:

price incl. = (price excl. + 0.10880) × 1.21

Here:  
- `price_excl.` is the dynamic hourly electricity price (€/kWh) from jeroen.nl.  
- `0.10880` represents the combined energy tax and surcharge in €/kWh.  
- `1.21` accounts for 21% VAT applied to the subtotal.

The energy tax values are obtained from the official Dutch Tax Authority  
([Belastingdienst](https://www.belastingdienst.nl/wps/wcm/connect/bldcontentnl/belastingdienst/zakelijk/overige_belastingen/belastingen_op_milieugrondslag/energiebelasting/)).

This adjustment ensures that both the fixed and dynamic price datasets are expressed in the same way.
