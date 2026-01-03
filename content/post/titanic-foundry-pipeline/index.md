+++
title = "[Palantir Foundry] Titanic Survivor Prediction Project (1/2) — Data Preprocessing"
description = "Palantir Foundry의 Pipeline Builder(노코드)로 Kaggle Titanic 데이터를 업로드하고 결측치 처리 및 피처 엔지니어링까지 진행해 모델링 준비를 마칩니다."
date = 2026-01-03T00:00:00+09:00
draft = false
slug = "titanic-foundry-pipeline-builder"
tags = ["Palantir","Foundry", "Titanic", "Data", "No-code"]
categories = ["Palantir"]
+++

In this post, we’ll use Kaggle’s classic beginner dataset—**Titanic: Machine Learning from Disaster**—and walk through how to create a project in **Palantir Foundry** and preprocess the data using **Pipeline Builder** (no-code) up to the point where it’s ready for modeling.

## Table of contents

- [1. Create and configure a project](#1-create-and-configure-a-project)
  - [1.1 Create a new project](#11-create-a-new-project)
- [2. Upload the data](#2-upload-the-data)
- [3. Preprocess data with Pipeline Builder](#3-preprocess-data-with-pipeline-builder)
- [4. Handle missing values (Age)](#4-handle-missing-values-age)
  - [4.1 Compute the mean age](#41-compute-the-mean-age)
  - [4.2 Join the mean back to the original rows](#42-join-the-mean-back-to-the-original-rows)
  - [4.3 Fill Null Age with Mean_Age](#43-fill-null-age-with-mean_age)
  - [4.4 Drop the temporary Mean_Age column](#44-drop-the-temporary-mean_age-column)
- [5. Handle missing values (Embarked)](#5-handle-missing-values-embarked)
- [6. Feature engineering](#6-feature-engineering)
  - [6.1 Family size](#61-family-size)
  - [6.2 Extract title from name](#62-extract-title-from-name)
  - [6.3 Encode categorical values (Sex)](#63-encode-categorical-values-sex)
- [7. Write out the cleaned dataset](#7-write-out-the-cleaned-dataset)
- [Wrap-up](#wrap-up)

## 1. Create and configure a project

### 1.1 Create a new project

First, create a workspace for this work.

- Click **New project** to start creating a project.
  ![New project](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228212257.png)
- On the template selection screen, choose **Production project** (recommended for collaboration and access control).
  ![Production project template](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228212403.png)
- Set the project name to something clear, e.g. `Titanic`, and create the project.
  ![Project name](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228212441.png)

## 2. Upload the data

Once the project is created, bring in the data you want to analyze. From the Kaggle Titanic competition page, download the following three files:

- `train.csv`
- `test.csv`
- `gender_submission.csv`

Kaggle competition page: [Titanic - Machine Learning from Disaster](https://www.kaggle.com/competitions/titanic)

In your Foundry project:

- Click **+ New**
  ![New button](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228212844.png)
- Click **Upload files** and upload all three files.
  ![Upload files](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228212949.png)
- When prompted for the data format, select **Upload as individual structured datasets (recommended)**. This converts CSV (structured) files into Foundry datasets that are immediately usable.
  ![Upload as structured datasets](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228213003.png)

## 3. Preprocess data with Pipeline Builder

Now it’s time to transform the data. We’ll use **Pipeline Builder**, which lets you build logic without writing code.

- Click **New**
  - ![New pipeline](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228213256.png)
- Select **Pipeline Builder**.
  - ![Pipeline Builder](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228213334.png)
- Keep the defaults (**Batch pipeline**, **Standard** mode) and click **Create pipeline**.
  - ![Create pipeline](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228213513.png)

Next, add your input dataset:

- Click **Add Foundry data**
  - ![Add Foundry data](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228213559.png)
- Select the uploaded `train` dataset.
  - ![Select train dataset](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228213824.png)
- Click **Add data**
  - ![Add data](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228213832.png)

## 4. Handle missing values (Age)

If you inspect the data, you’ll notice missing (Null) values in the `Age` column. Instead of dropping those rows, we’ll fill missing `Age` values using the **overall mean age**.

![Missing Age](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228214258.png)

### 4.1 Compute the mean age

- From the `train` node, choose **Transform**.
  - ![Transform](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228214355.png)
- Click **Aggregate**.
  - ![Aggregate](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228214418.png)
- Set Aggregations: ‘Mean’, Expression: `Age`, Output: `Mean_Age` and click `Apply`.
  - ![Apply aggregate](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228214548.png)
- Confirm the output, then `Close`.
  - ![Aggregate result](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228214639.png)

### 4.2 Join the mean back to the original rows

Now we need to attach the computed mean age (≈ 29.7) to each row in the original dataset.

- Select the `train` node, then click **Join**.
  - ![Join](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228214844.png)
- Click **Transform path** → **Start** (Left: `train`, Right: `Transform path`)
  - ![Transform path start](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228215006.png)
- Set **Join type** to **Cross join**, then click **Apply** and **Close**.
  - This appends the same `Mean_Age` value to every row.
  - ![Cross join](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228215106.png)

You should see `Mean_Age` added at the far right of the table:

![Mean_Age column](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228215207.png)

### 4.3 Fill Null `Age` with `Mean_Age`

We’ll create logic that says: If `Age` is null → use `Mean_Age`, Else → keep the original `Age`


- From the Join node, click **Transform**.
  - ![Transform after join](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228215400.png)
- Choose **Case**.
  - ![Case transform](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228215509.png)
- Condition: `Is null`, **Expression**: `Age`
  - ![Case apply](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228221755.png)
- true(next to ‘is equal to’), **Then**: `Mean_Age`, **Else**: `Age`, Click **Apply**.
  - ![Case result](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228215717.png)

Apply this so the `Age` column is overwritten with the filled value.

### 4.4 Drop the temporary `Mean_Age` column

After filling, `Mean_Age` is no longer needed. To keep the dataset clean:

- Use **Apply Multiple expressions** to exclude `Mean_Age` and keep the remaining columns.
  - ![Apply multiple expressions](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228222008.png)
- Click **Add item**, select everything except `Mean_Age`, uncheck **Keep remaining columns**, then click **Apply**.
  - ![Exclude Mean_Age](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228222158.png)

## 5. Handle missing values (Embarked)

When you check the distribution of `Embarked`, you’ll typically find that `S` (Southampton) is the most frequent value. We’ll fill missing `Embarked` values with the mode: `S`.

![Embarked distribution](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228223012.png)

- Choose **Case**.
  ![Embarked case](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228223104.png)
- Condition: `Embarked` **Is null**
- **Then**: `"S"` (a literal string)
- **Else**: `Embarked`
- Click **Apply**.
  ![Embarked apply](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228223322.png)

## 6. Feature engineering

To improve downstream model performance, let’s create a few additional columns from existing data.

### 6.1 Family size

- `SibSp`: number of siblings/spouses aboard the Titanic
- `Parch`: number of parents/children aboard the Titanic

we can estimate how many family members were traveling together. We’ll also add `1` to include the passenger themself.

- Click **Add numbers**.
  - ![Add numbers](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228223537.png)
- **Expressions**: `SibSp`, `Parch`, `1`
- **Output**: `FamilySize`
- Click **Apply**.
  - ![FamilySize result](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228223840.png)

### 6.2 Extract title from name

We can extract the honorific (e.g., `Mr`, `Mrs`, `Miss`) from the `Name` field using a regex.

- Click **Regex extract**.
  - ![Regex extract](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228224026.png)
- **Expression**: `Name`
- **Pattern**: `([A-Za-z]+)\.`
- **Group**: `1`
- **Output**: `Title`
- Click **Apply**.
 ![Regex Apply](/blog/p/titanic-foundry-pipeline-builder//Pasted%20image%2020251229232426.png)

### 6.3 Encode categorical values (Sex)

Machine learning models typically work better with numeric features than raw strings. Let’s convert `Sex` (`male`, `female`) into a numeric column.

- Choose **Case**.
  - ![Sex case](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228224742.png)
- If `Sex == "male"` → `1`
- If `Sex == "female"` → `0`
- Else → `Null`
- Set **Output** to `Sex_Encoded`, then click **Apply**.
  - ![Sex encoded](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228224850.png)

## 7. Write out the cleaned dataset

Once preprocessing is complete, save the final dataset for modeling.

- Click **Add output**.
  - ![Add output](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228225248.png)
- Click **New dataset**.
  - ![New dataset](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228225311.png)
- Set the dataset name to `titanic_cleaned_train`.
  - ![Dataset name](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228225542.png)
- Click the green upward arrow (save all changes).
  - ![Save changes](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228225506.png)
- Click **Deploy** → **Deploy pipeline**.
  - ![Deploy menu](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228225614.png)
  - ![Deploy pipeline](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228225652.png)

After a short wait, the pipeline deployment should complete successfully (`Successfully deployed pipeline`), and you’ll have a clean, processed dataset ready for training.

![Deploy success](/blog/p/titanic-foundry-pipeline-builder/Pasted%20image%2020251228225820.png)

## Wrap-up

Today we used Pipeline Builder to preprocess the Titanic dataset without coding: filling missing values, creating derived features, and encoding categorical data. In the next post, we’ll take the resulting `titanic_cleaned_train` dataset and move on to **training a machine learning model and visualizing survival predictions (Workshop)**.


