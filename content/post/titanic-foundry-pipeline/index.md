+++
title = "Titanic 데이터 전처리: Foundry Pipeline Builder로 결측치 처리 & 피처 엔지니어링"
description = "Palantir Foundry의 Pipeline Builder(노코드)로 Kaggle Titanic 데이터를 업로드하고 결측치 처리 및 피처 엔지니어링까지 진행해 모델링 준비를 마칩니다."
date = 2026-01-03T00:00:00+09:00
draft = false
slug = "titanic-foundry-pipeline-builder"
tags = ["Foundry", "Titanic", "Data", "No-code"]
categories = ["Data"]
+++

In this post, we’ll use Kaggle’s classic beginner dataset—**Titanic: Machine Learning from Disaster**—and walk through how to create a project in **Palantir Foundry** and preprocess the data using **Pipeline Builder** (no-code) up to the point where it’s ready for modeling.

## 1. Create and configure a project

### 1.1 Create a new project

First, create a workspace for this work.

- Click **New project** to start creating a project.
  - (Insert screenshot: `Pasted image 20251228212257`)
- On the template selection screen, choose **Production project** (recommended for collaboration and access control).
  - (Insert screenshot: `Pasted image 20251228212403`)
- Set the project name to something clear, e.g. `Titanic`, and create the project.
  - (Insert screenshot: `Pasted image 20251228212441`)

## 2. Upload the data

Once the project is created, bring in the data you want to analyze. From the Kaggle Titanic competition page, download the following three files:

- `train.csv`
- `test.csv`
- `gender_submission.csv`

Kaggle competition page: [Titanic - Machine Learning from Disaster](https://www.kaggle.com/competitions/titanic)

In your Foundry project:

- Click **+ New**
  - (Insert screenshot: `Pasted image 20251228212844`)
- Click **Upload files** and upload all three files.
  - (Insert screenshot: `Pasted image 20251228212949`)
- When prompted for the data format, select **Upload as individual structured datasets (recommended)**.
  - This converts CSV (structured) files into Foundry datasets that are immediately usable.
  - (Insert screenshot: `Pasted image 20251228213003`)

## 3. Preprocess data with Pipeline Builder

Now it’s time to transform the data. We’ll use **Pipeline Builder**, which lets you build logic without writing code.

- Click **New**
  - (Insert screenshot: `Pasted image 20251228213256`)
- Select **Pipeline Builder**.
  - (Insert screenshot: `Pasted image 20251228213334`)
- Keep the defaults (**Batch pipeline**, **Standard** mode) and click **Create pipeline**.
  - (Insert screenshot: `Pasted image 20251228213513`)

Next, add your input dataset:

- Click **Add Foundry data**
  - (Insert screenshot: `Pasted image 20251228213559`)
- Select the uploaded `train` dataset.
  - (Insert screenshot: `Pasted image 20251228213824`)
- Click **Add data**
  - (Insert screenshot: `Pasted image 20251228213832`)

## 4. Handle missing values (Age)

If you inspect the data, you’ll notice missing (Null) values in the `Age` column. Instead of dropping those rows, we’ll fill missing `Age` values using the **overall mean age**.

- (Insert screenshot: `Pasted image 20251228214258`)

### 4.1 Compute the mean age

- From the `train` node, choose **Transform**.
  - (Insert screenshot: `Pasted image 20251228214355`)
- Click **Aggregate**.
  - (Insert screenshot: `Pasted image 20251228214418`)
- Configure:
  - **Aggregations**: `Mean`
  - **Expression**: `Age`
  - **Output**: `Mean_Age`
- Click **Apply**, confirm the output, then **Close**.
  - (Insert screenshot: `Pasted image 20251228214548`)
  - (Insert screenshot: `Pasted image 20251228214639`)

### 4.2 Join the mean back to the original rows

Now we need to attach the computed mean age (≈ 29.7) to each row in the original dataset.

- Select the `train` node, then click **Join**.
  - (Insert screenshot: `Pasted image 20251228214844`)
- Choose the transform output as the right side:
  - Click **Transform path** → **Start** (Left: `train`, Right: `Transform path`)
  - (Insert screenshot: `Pasted image 20251228215006`)
- Set **Join type** to **Cross join**, then click **Apply** and **Close**.
  - This appends the same `Mean_Age` value to every row.
  - (Insert screenshot: `Pasted image 20251228215106`)

You should see `Mean_Age` added at the far right of the table:

- (Insert screenshot: `Pasted image 20251228215207`)

### 4.3 Fill Null `Age` with `Mean_Age`

We’ll create logic that says:

- If `Age` is null → use `Mean_Age`
- Else → keep the original `Age`

Steps:

- From the Join node, click **Transform**.
  - (Insert screenshot: `Pasted image 20251228215400`)
- Choose **Case**.
  - (Insert screenshot: `Pasted image 20251228215509`)
- Configure:
  - Condition: **Is null**
  - **Expression**: `Age`
  - **Then**: `Mean_Age`
  - **Else**: `Age`
- Click **Apply**.
  - (Insert screenshot: `Pasted image 20251228221755`)
  - (Insert screenshot: `Pasted image 20251228215717`)

Apply this so the `Age` column is overwritten with the filled value.

### 4.4 Drop the temporary `Mean_Age` column

After filling, `Mean_Age` is no longer needed. To keep the dataset clean:

- Use **Apply Multiple expressions** to exclude `Mean_Age` and keep the remaining columns.
  - (Insert screenshot: `Pasted image 20251228222008`)
- Click **Add item**, select everything except `Mean_Age`, uncheck **Keep remaining columns**, then click **Apply**.
  - (Insert screenshot: `Pasted image 20251228222158`)

## 5. Handle missing values (Embarked)

When you check the distribution of `Embarked`, you’ll typically find that `S` (Southampton) is the most frequent value. We’ll fill missing `Embarked` values with the mode: `S`.

- (Insert screenshot: `Pasted image 20251228223012`)

Steps:

- Choose **Case**.
  - (Insert screenshot: `Pasted image 20251228223104`)
- Configure:
  - Condition: `Embarked` **Is null**
  - **Then**: `"S"` (a literal string)
  - **Else**: `Embarked`
- Click **Apply**.
  - (Insert screenshot: `Pasted image 20251228223322`)

## 6. Feature engineering

To improve downstream model performance, let’s create a few additional columns from existing data.

### 6.1 Family size

By combining:

- `SibSp`: number of siblings/spouses aboard the Titanic
- `Parch`: number of parents/children aboard the Titanic

…we can estimate how many family members were traveling together. We’ll also add `1` to include the passenger themself.

- Click **Add numbers**.
  - (Insert screenshot: `Pasted image 20251228223537`)
- Set:
  - **Expressions**: `SibSp`, `Parch`, `1`
  - **Output**: `FamilySize`
- Click **Apply**.
  - (Insert screenshot: `Pasted image 20251228223840`)

### 6.2 Extract title from name

We can extract the honorific (e.g., `Mr`, `Mrs`, `Miss`) from the `Name` field using a regex.

- Click **Regex extract**.
  - (Insert screenshot: `Pasted image 20251228224026`)
- Set:
  - **Expression**: `Name`
  - **Pattern**: `([A-Za-z]+)\.`
  - **Group**: `1`
  - **Output**: `Title`
- Click **Apply**.

### 6.3 Encode categorical values (Sex)

Machine learning models typically work better with numeric features than raw strings. Let’s convert `Sex` (`male`, `female`) into a numeric column.

- Choose **Case**.
  - (Insert screenshot: `Pasted image 20251228224742`)
- Create two cases:
  - If `Sex == "male"` → `1`
  - If `Sex == "female"` → `0`
  - Else → `Null`
- Set **Output** to `Sex_Encoded`, then click **Apply**.
  - (Insert screenshot: `Pasted image 20251228224850`)

## 7. Write out the cleaned dataset

Once preprocessing is complete, save the final dataset for modeling.

- Click **Add output**.
  - (Insert screenshot: `Pasted image 20251228225248`)
- Click **New dataset**.
  - (Insert screenshot: `Pasted image 20251228225311`)
- Set the dataset name to `titanic_cleaned_train`.
  - (Insert screenshot: `Pasted image 20251228225542`)
- Click the green upward arrow (save all changes).
  - (Insert screenshot: `Pasted image 20251228225506`)
- Click **Deploy** → **Deploy pipeline**.
  - (Insert screenshot: `Pasted image 20251228225614`)
  - (Insert screenshot: `Pasted image 20251228225652`)

After a short wait, the pipeline deployment should complete successfully (`Successfully deployed pipeline`), and you’ll have a clean, processed dataset ready for training.

- (Insert screenshot: `Pasted image 20251228225820`)

## Wrap-up

Today we used Pipeline Builder to preprocess the Titanic dataset without coding: filling missing values, creating derived features, and encoding categorical data. In the next post, we’ll take the resulting `titanic_cleaned_train` dataset and move on to **training a machine learning model and visualizing survival predictions (Workshop)**.


