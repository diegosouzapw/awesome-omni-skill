---
name: Logistics Cost Optimization
description: Identify cost-saving opportunities across transportation, warehousing, and last-mile delivery by analyzing cost-per-unit, mode optimization, network design efficiency, and carrier performance benchmarks.

metadata:
  display_name: "Logistics Cost Optimization"
  short_description: "Optimize logistics costs across transport and warehousing"
  default_prompt: "Optimize my logistics cost and suggest the best next steps"
  version: "1.0.0"
  tags:
    - cpg-retail
  icon_path: "assets/icon.png"
---

# Logistics Cost Optimization

## Overview

This skill analyzes the full logistics cost structure—inbound transportation, outbound distribution, warehousing, and last-mile delivery—to identify actionable cost-saving opportunities. It benchmarks costs against industry standards, evaluates mode and carrier optimization, assesses network design efficiency, and quantifies savings from consolidation, route optimization, and contract renegotiation. The goal is to reduce total landed cost while maintaining or improving service levels.

## When to Use

- During annual logistics budget planning and carrier contract negotiations
- When transportation costs as a percentage of revenue exceed industry benchmarks
- After network changes (new DC, store openings/closings, supplier changes)
- When fuel surcharges or carrier rate increases erode margins
- During quarterly logistics performance reviews
- When evaluating insource vs. outsource (3PL) decisions
- Before peak season to optimize capacity procurement strategy

## Required Inputs

| Input | Description | Format |
|-------|-------------|--------|
| `shipment_data` | Historical shipments with origin, destination, mode, carrier, weight, cube, cost | Structured array (12+ months) |
| `freight_invoices` | Detailed freight bills with line-item charges (linehaul, fuel, accessorial) | Structured array |
| `warehouse_costs` | Storage, labor, equipment, overhead by facility | Structured object |
| `carrier_contracts` | Rate tables, discount tiers, minimum commitments | Structured object |
| `order_data` | Order profiles (size, frequency, delivery requirements) | Structured array |
| `network_topology` | DC locations, store/customer locations, supplier locations | Geographic data |
| `service_requirements` | Transit time targets, delivery windows, temperature requirements | Structured object |

## Methodology

### Step 1: Total Logistics Cost Decomposition

Break down logistics costs into controllable components:

```
Total_Logistics_Cost = Inbound_Transportation + Outbound_Transportation 
                     + Warehousing_Cost + Last_Mile_Delivery + Returns_Logistics
                     + Inventory_Carrying_Cost_in_Transit
```

Calculate key cost ratios:
```
Logistics_Cost_as_%_Revenue = Total_Logistics_Cost / Net_Revenue × 100
Transportation_Cost_per_Unit = Total_Transport_Cost / Total_Units_Shipped
Cost_per_Order = Total_Logistics_Cost / Total_Orders
Cost_per_Pound = Freight_Cost / Total_Weight_Shipped
Warehouse_Cost_per_Unit = Total_Warehouse_Cost / Total_Units_Throughput
```

Industry benchmarks (CPG retail):
- Logistics as % of revenue: 6-10% (grocery), 8-14% (general merchandise)
- Transportation as % of logistics: 55-65%
- Warehousing as % of logistics: 25-35%

### Step 2: Transportation Mode Optimization

Evaluate mode mix for cost-service trade-offs:

| Mode | Cost Index | Transit Time | Best For |
|------|-----------|-------------|----------|
| Ocean (FCL) | 1.0× | 25-45 days | High-volume, long lead time, non-urgent |
| Ocean (LCL) | 1.5× | 30-50 days | Low volume international, consolidation |
| Rail (intermodal) | 2-3× | 5-10 days | Domestic long-haul, heavy goods |
| Truckload (FTL) | 4-6× | 1-5 days | Full loads, time-sensitive domestic |
| Less-than-truckload (LTL) | 8-12× | 2-7 days | Partial loads, multi-stop |
| Air freight | 15-25× | 1-3 days | Emergency, perishable, high-value-to-weight |
| Parcel | Varies | 1-5 days | Small shipments, e-commerce DTC |

**Mode shift opportunity identification**:
```
Mode_Shift_Savings = Σ(Shipments_Eligible × (Current_Mode_Cost - Alternative_Mode_Cost))
Subject to: Alternative_Transit_Time ≤ Required_Transit_Time
```

Flag shipments where air freight was used but ocean/ground would have met the delivery window — these are emergency expedites that indicate planning failures.

### Step 3: Carrier Performance & Rate Analysis

Benchmark carrier costs and performance:

```
Carrier_Cost_Index = Carrier_Rate / Lane_Benchmark_Rate
Carrier_Service_Score = On_Time_Delivery% × 0.5 + Damage_Free% × 0.3 + Claims_Resolution_Speed × 0.2
Value_Score = Service_Score / Cost_Index  [higher = better value]
```

Identify optimization opportunities:
- **Rate shopping**: For each lane, identify if the lowest-cost carrier meeting service requirements is being used
- **Volume consolidation**: Are shipments split across carriers where consolidating would hit discount tiers?
- **Contract compliance**: Are contracted rates being applied, or is spot market leakage occurring?
- **Accessorial reduction**: Analyze detention, demurrage, redelivery, and liftgate charges for reduction opportunities

### Step 4: Shipment Consolidation Analysis

Identify consolidation opportunities to improve equipment utilization:

```
Truck_Utilization = Actual_Weight_or_Cube / Truck_Capacity × 100
Average_Utilization = Σ(Shipment_Utilization) / Total_Shipments
```

Target utilization: >85% for FTL, >70% for LTL

Consolidation strategies:
- **Temporal consolidation**: Delay shipments by 1-2 days to build full loads (trade-off: transit time)
- **Geographic consolidation**: Pool shipments to nearby destinations into multi-stop routes
- **Cross-customer consolidation**: For 3PL or shared distribution, combine orders across customers
- **Pool distribution**: Ship FTL to a regional pool point, then break-bulk for final delivery

```
Consolidation_Savings = (Current_LTL_Cost - Equivalent_FTL_Cost_per_Stop) × Eligible_Shipments
```

### Step 5: Warehouse Cost Optimization

Analyze warehouse cost drivers:

```
Storage_Cost_per_Pallet_per_Month = (Rent + Utilities + Insurance) / Average_Pallet_Positions
Labor_Cost_per_Unit = Total_Warehouse_Labor / Units_Picked_and_Shipped
Warehouse_Utilization = Peak_Positions_Used / Total_Available_Positions
Throughput_Efficiency = Units_Shipped / Labor_Hours
```

Optimization levers:
- **Slotting optimization**: Move fast-movers to prime pick locations (see warehouse-slotting-optimizer skill)
- **Labor scheduling**: Align labor hours with inbound/outbound volume waves
- **Automation ROI**: Calculate payback on automation investments (AS/RS, goods-to-person, automated sortation)
- **Space utilization**: Evaluate rack density, aisle width optimization, mezzanine additions

### Step 6: Network Design Efficiency

Assess if the distribution network is optimally configured:

```
Average_Distance_to_Customer = Σ(Shipment_Distance × Shipment_Volume) / Total_Volume
Network_Coverage = % of Customers within 1-day/2-day ground transit
Transportation_Cost_Gradient = Cost_per_Unit at varying DC counts (1, 2, 3, ... N DCs)
```

Trade-off analysis:
- More DCs = lower outbound transport + faster delivery BUT higher fixed costs + more inventory
- Fewer DCs = lower fixed costs + inventory pooling BUT higher transport + slower delivery
- Optimal DC count minimizes `Total_Transport_Cost + Total_Inventory_Cost + Total_Facility_Cost`

## Output Specification

```yaml
logistics_optimization_report:
  analysis_period: "2025-02 to 2026-01"
  total_logistics_spend: 28500000
  logistics_as_pct_revenue: 9.2
  benchmark_target: 7.8
  gap_value: 4340000
  opportunities:
    - id: "OPT-001"
      category: "Mode Optimization"
      description: "Shift 340 air shipments to ocean+ground where lead time permits"
      current_annual_cost: 2800000
      optimized_annual_cost: 1150000
      annual_savings: 1650000
      implementation_effort: "medium"
      timeline: "8-12 weeks"
      risk: "Requires 3-week additional lead time planning buffer"
    - id: "OPT-002"
      category: "Shipment Consolidation"
      description: "Consolidate LTL shipments to Southeast region into 2× weekly FTL"
      current_annual_cost: 1200000
      optimized_annual_cost: 780000
      annual_savings: 420000
      implementation_effort: "low"
      timeline: "4 weeks"
    - id: "OPT-003"
      category: "Carrier Optimization"
      description: "Shift 60% of Midwest FTL volume to Carrier B (15% lower rate, same service)"
      annual_savings: 380000
      implementation_effort: "low"
      timeline: "2 weeks (contract amendment)"
    - id: "OPT-004"
      category: "Accessorial Reduction"
      description: "Reduce detention charges through dock scheduling system implementation"
      current_accessorial_cost: 450000
      target_reduction: 280000
      implementation_effort: "medium"
      timeline: "12 weeks"
  total_identified_savings: 2730000
  savings_as_pct_spend: 9.6
```

## Analysis Framework

### Pareto Analysis of Cost Drivers

Apply 80/20 analysis across dimensions:
- **By lane**: Top 20% of lanes typically represent 70-80% of freight spend
- **By carrier**: Often 3-5 carriers represent the majority of spend
- **By accessorial type**: Fuel surcharges and detention usually dominate accessorial costs
- **By shipment size**: Identify the cost penalty for small, inefficient shipments

### Total Landed Cost Model

```
Landed_Cost = Product_Cost + Inbound_Freight + Customs_Duties + Warehousing_Allocation 
            + Outbound_Freight + Last_Mile + Returns_Cost + Inventory_Carrying
```

Compare landed cost across sourcing and fulfillment scenarios to optimize end-to-end, not just transport.

## Examples

**Example 1 — Air-to-Ocean Mode Shift**
> "Analysis of 12 months of shipments reveals 340 air freight shipments ($2.8M) from Southeast Asian suppliers where order lead time was sufficient for ocean transit. Root cause: late purchase order placement forcing air expedite. By improving PO timing by 3 weeks, 85% of these shipments can move to ocean+ground at $1.15M — annual savings of $1.65M. Prerequisite: upstream planning discipline and 3-week safety stock buffer increase ($220K carrying cost)."

**Example 2 — LTL Consolidation**
> "Southeast region receives an average of 12 LTL shipments per week averaging 6,200 lbs each (38% truck utilization). Consolidating into 2 FTL shipments per week (37,200 lbs, 93% utilization) reduces annual cost from $1.2M to $780K. Customer service impact: delivery window shifts from daily to Mon/Thu, requiring customer communication. For time-sensitive customers (15% of volume), maintain daily LTL at current rates."

## Guidelines

1. Always evaluate cost optimization against service level impact — the cheapest option that misses delivery windows is not an optimization
2. Focus on the top 20 lanes first — they represent the majority of spend and savings potential
3. Separate structural savings (mode shift, network redesign) from tactical savings (rate negotiation, consolidation)
4. Include implementation costs and timeline in savings projections — net present value, not just gross savings
5. Benchmark against industry indices (DAT, Freightos, Cass Freight Index) not just internal history
6. Account for seasonality in cost analysis — peak season rates can be 20-40% higher than off-peak
7. Track cost per unit shipped over time as the primary trend metric, not total spend (which fluctuates with volume)
8. Evaluate 3PL vs. private fleet economics at least annually using fully loaded cost comparison
9. Consider carbon footprint alongside cost — some cost optimizations (mode shift to rail) also reduce emissions

## Validation Checklist

- [ ] Freight invoice data has been audited for billing errors (industry average: 3-5% error rate)
- [ ] Cost benchmarks use current market rates, not stale contract rates
- [ ] Mode shift analysis confirms transit time feasibility against actual customer requirements
- [ ] Consolidation analysis accounts for product compatibility (e.g., food cannot ship with chemicals)
- [ ] Carrier rate comparisons use normalized basis (same accessorials, fuel surcharge methodology)
- [ ] Warehouse cost allocation methodology is consistent and defensible
- [ ] Network optimization accounts for inventory positioning costs, not just transportation
- [ ] Savings estimates are net of implementation costs and ongoing operational changes
- [ ] Service level constraints are explicitly stated and validated with commercial/sales teams
- [ ] Recommendations have been reviewed with logistics operations for practical feasibility
