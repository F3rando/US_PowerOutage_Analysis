## Illuminating the Grid: A Predictive Analysis of U.S. Power Outage Resilience

**By: Luis Corona**

## Introduction

Power outages are more than a temporary inconvenience, they are a critical indicator of a nation's infrastructure resilience. In an era where extreme weather events are becoming more frequent, understanding how and why our electrical grids fail is essential for effective urban planning and emergency response.

This project investigates a dataset of major power outages in the United States, seeking to answer one central question:

> **What factors—specifically related to geography, climate, and population density—are the most significant predictors of power outage duration?**

## The Dataset

The data utilized in this analysis contains 1,534 rows, each representing a major power outage in the U.S. occurring between 2000 and 2016. By analyzing these records, I aim to identify patterns in how different regions respond to and recover from grid failures.

## Relevant Columns

To answer my research question and build a predictive model, I focused on the following key features:

- **`OUTAGE.DURATION` (Minutes)**: My target variable, the total time from the start of the outage until power was fully restored.
- **`CLIMATE.REGION`**: The geographical area where the outage occurred (e.g., Northeast, Central).
- **`CAUSE.CATEGORY`**: The primary reason for the outage (e.g., Severe Weather, Equipment Failure).
- **`ANOMALY.LEVEL`**: A quantitative measure of climate anomalies (like El Niño/La Niña) present during the outage.
- **`POPDEN_URBAN`**: The population density of urban areas in the affected state, which serves as a proxy for infrastructure complexity and repair priority.