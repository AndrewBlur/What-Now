supervised regression problem -> Goal -> Predict geological position of a drill bit while drilling a horizontal oil well


# What is TVT?

The target variable is **TVT (True Vertical Thickness/True Vertical Position depending on the competition definition).**

Think of it as

> "How far vertically is the drill from a geological reference?"

Example

```
Reservoir Top
────────────────────

+4 m

Drill Bit

0 m

Target Layer

-3 m

Reservoir Bottom
────────────────────
```

Positive and negative values indicate the drill's vertical position relative to the formation.

# Input

The dataset contains measurements collected while drilling.

Typically these include

- Gamma Ray
- Resistivity
- Inclination
- Azimuth
- Weight on Bit
- Rate of Penetration
- Torque
- Mud properties
- Well trajectory
- Previous measurements

Each row is one point along the horizontal well.

# Output

For every row in the test set

```
id,tvt
000d7d20_1442,2.31
000d7d20_1443,2.28
000d7d20_1444,2.40
```


# Evaluation Metric

The competition uses **Root Mean Squared Error (RMSE).**

Formula

$$RMSE=\sqrt{\frac1n\sum_{i=1}^{n}(y_i-\hat y_i)^2}$$​

where

- $y_i$​ = true TVT
- $\hat y_i$ = predicted TVT

Lower RMSE is better.