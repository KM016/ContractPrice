# Natural Gas Storage Contract Pricing

A prototype Python function for valuing simplified natural gas storage opportunities using estimated injection and withdrawal prices.


> 09/2024<br>
> JPMorgan Chase Forage Quantitative Research Job Simulation


# Natural Gas Storage Contract Pricing

A prototype Python function for valuing simplified natural gas storage opportunities using estimated injection and withdrawal prices.


## Project overview

Natural gas storage can create value when gas is bought and injected at a lower price, stored, and later withdrawn and sold at a higher price. This notebook extends the forecasting work from Task 1 and combines estimated prices with a simple set of storage assumptions.

The resulting `contract_price(...)` function accepts one or more injection and withdrawal periods and estimates their combined value after accounting for storage duration, volume, capacity and transport assumptions.

## Objectives

The task was to turn the earlier price-estimation work into a prototype that could:

- value several injection and withdrawal periods in one call;
- connect storage economics to date-specific gas prices;
- limit the stored volume using injection-rate and capacity constraints;
- account for the principal operating costs; and
- return both period-level values and a combined contract value.

## Price model

The first half of `task2.ipynb` reproduces the Task 1 analysis. It loads the 48 monthly prices, checks stationarity, fits a SARIMAX `(1, 1, 1) × (1, 1, 1, 12)` model and defines `estimate_price(date)`.

Observed month-end prices and forecast month-end prices are interpolated to daily dates. The storage function calls this estimator for every injection and withdrawal date supplied by the user.

## Inputs

The pricing function uses:

- injection and withdrawal dates;
- estimated natural gas prices on those dates;
- monthly injection rate;
- maximum storage volume;
- monthly storage cost;
- an injection/withdrawal cost parameter; and
- transport costs for moving gas into and out of storage.

For each date pair, the notebook estimates the volume that can be stored, calculates the purchase and sale cash flows, applies the stated cost assumptions and returns both the individual period values and their total.

## Implementation

The notebook contains two linked parts:

1. a seasonal SARIMAX model fitted to the supplied monthly natural gas prices; and
2. `contract_price(injectionDate, withdrawDate, storageCost, i_w_cost, maxVolume, rate, transpCost)`, which applies the simplified contract assumptions.

For each matched pair of dates, the function:

1. estimates the gas price at injection and withdrawal;
2. calculates the storage duration in approximate months using `days / 30.4`;
3. calculates the storage fee from duration and monthly storage cost;
4. sets volume to the lower of `months stored × injection rate` and maximum capacity;
5. calculates the gas purchase and sale cash flows;
6. applies the injection/withdrawal and two-way transport assumptions; and
7. appends the rounded period value to the output list.

The function first checks that the injection-date and withdrawal-date lists have equal lengths. If they do not, it returns an error rather than attempting a partial valuation.

## Worked example

The saved example uses:

- five storage periods between December 2019 and September 2025;
- an injection rate of 1,000,000 units per month;
- maximum capacity of 5,000,000 units;
- monthly storage cost of `$100,000`;
- injection/withdrawal cost of `$10,000` per million units; and
- transport cost of `$50,000` in each direction.

The demonstration returns five period values:

```text
-620,321.63
-950,657.89
1,676,818.05
6,883,749.26
-4,971,711.97
```

Their reported total is `$2,017,875.82`. This is the output of the preserved prototype and should be read alongside the assumptions and limitations below.

## Tools used

- pandas and NumPy for dates, data handling and calculations;
- Matplotlib for visualisation; and
- statsmodels for stationarity testing and SARIMAX forecasting.

## Repository contents

```text
.
├── Nat Gas.csv    # Monthly prices supplied with the simulation
├── task2.ipynb    # Price model and storage-contract prototype
└── README.md
```

## Scope and limitations

This is a simplified prototype rather than a production storage valuation engine. It does not construct a full inventory schedule, discount cash flows, optimise injection and withdrawal decisions, model bid-ask spreads or enforce every operational constraint found in a real storage agreement. It also inherits the forecasting limitations of Task 1. The treatment and sign convention of each cost parameter should be reviewed before the function is used outside the original exercise.
