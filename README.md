
# HR Employee Attrition Analysis

I'm looking at what actually drives employees to leave a company, and whether it's possible to predict who's at risk before they go.

## Dataset Content

The dataset I'm using is the IBM HR Analytics Employee Attrition & Performance dataset from Kaggle, a synthetic dataset of 1,470 employees with 35 columns covering demographics (Age, Gender, MaritalStatus, DistanceFromHome), compensation (MonthlyIncome, DailyRate, PercentSalaryHike), role and tenure (Department, JobRole, YearsAtCompany), and self-reported satisfaction ratings (JobSatisfaction, WorkLifeBalance, EnvironmentSatisfaction). The target column is Attrition (Yes/No). There were no missing values or duplicate rows in the raw data.

Source: https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset

## Business Requirements

1. **Attrition Overview** — What is the overall employee attrition rate and how does it vary across departments and job roles?
2. **Attrition Drivers** — Which employee and workplace factors are most strongly associated with employee attrition?
3. **Employee Satisfaction & Work Conditions** — How do job satisfaction, work-life balance, overtime and other workplace conditions relate to employee attrition?
4. **Attrition Prediction** — Can employee attrition be predicted using employee demographic, employment and workplace characteristics?

## Hypotheses and How to Validate

**Overtime is associated with employee attrition.**
- Validation: chi-square test of independence between OverTime and Attrition
- Result: Chi2 = 87.56, p < 0.001
- Summary: Employees who work overtime attrite at 30.5%, nearly three times the rate of employees who don't (10.4%). This is by far the largest effect of any hypothesis tested.

**There is a difference in Monthly Income between employees who leave and those who stay.**
- Validation: t-test comparing MonthlyIncome between attrition groups
- Result: T = -7.483, p < 0.001
- Summary: Employees who leave earn noticeably less on average (mean 4,787, median 3,202) than employees who stay (mean 6,833, median 5,204).

**There is a difference in Job Satisfaction between employees who leave and those who stay.**
- Validation: t-test comparing JobSatisfaction between attrition groups
- Result: T = -3.926, p < 0.001
- Summary: Employees who leave report lower job satisfaction on average (mean 2.47) than employees who stay (mean 2.78). The effect is real but smaller than the Overtime or Income results. This finding was consistent across multiple statistical tests (chi-square, Student's t-test, Welch's t-test, and Mann-Whitney U), all returning p < 0.001, confirming it's a robust result rather than something dependent on the choice of test.

**Department is associated with employee attrition.**
- Validation: chi-square test of independence between Department and Attrition
- Result: Chi2 = 10.80, p = 0.005
- Summary: Sales (20.6%) and HR (19.0%) attrite more than R&D (13.8%). This is a real, statistically significant effect, but the smallest of the four tested. I added this hypothesis myself to cover Business Requirement 1 (attrition variation by department/role), which wasn't addressed by the first three hypotheses my partner and I agreed on.

## Project Plan

## Project Structure

```text
employee-attrition-hackathon/
│
├── data/
│   ├── raw/
│   └── clean/
│
├── jupyter_notebooks/
│   ├── 01-ETL.ipynb
│   ├── 02-EDA.ipynb
│   ├── 03-Visualisation.ipynb
│   └── 04-ML.ipynb
│
├── Dashboard/
│   ├── 01-employee-attrition-overview.png
│   ├── 02-attrition-drivers.png
│   ├── 03-predictive-insight.png
│   └── Employee-Attrition-Dashboard.pbix
│
├── requirements.txt
└── README.md

```
Kanban board:

## Rationale — Business Requirements to Visualisations

| Business Requirement | Addressed By |
|---|---|
| 1. Attrition Overview | EDA notebook — overall rate, department and job role breakdown |
| 2. Attrition Drivers | All four hypotheses individually, plus the feature importance ranking in the Modelling notebook |
| 3. Satisfaction & Work Conditions | Overtime and Job Satisfaction hypotheses |
| 4. Attrition Prediction | Modelling notebook — Logistic Regression / Random Forest comparison |

## Analysis Techniques Used

Descriptive statistics (mean, median, std by group), normality testing (Shapiro-Wilk via pingouin), t-tests and chi-square tests of independence, correlation analysis, Logistic Regression and Random Forest classification with a scikit-learn Pipeline and ColumnTransformer, feature importance ranking, and a fairness check by subgroup. I used precision and recall alongside accuracy when evaluating the models, since Attrition is imbalanced (84% stayed vs 16% left) and accuracy alone would be misleading, this actually came up directly in the Modelling notebook, where the single-feature baseline models scored almost identically to just predicting "stayed" for everyone.

## Ethical Considerations

**Data Privacy and Governance**
This dataset is IBM's synthetic sample, but it represents the kind of sensitive personal and performance data (compensation, satisfaction, tenure) that a real HR system would hold. Any real-world use of an analysis like this would need to handle that data under standard employee data protection principles, access limited to those with a legitimate need, and clear retention and deletion policies.

**Bias and Fairness in the Data**
MaritalStatus shows a real spread in the raw data, single employees attrite at 25.0% vs 10.1% for divorced employees. Age also differs between groups, leavers average younger. Neither of these were tested as formal hypotheses, but they're flagged here since they showed up clearly in the data itself.

**Algorithmic Fairness**
The fairness check by subgroup in the Modelling notebook found the Random Forest model performs noticeably less accurately for Single employees than for Divorced employees, a gap of over 13 percentage points. This ties back to Single employees having a higher underlying attrition rate to begin with (more genuinely hard cases for the model to get right), rather than an obvious flaw in the model itself, but it's still a real limitation worth being upfront about before this model is used for any actual decision-making.

**Responsible Model Use**
Individual "flight risk" scores from a model like this shouldn't be shown directly to a line manager. Doing so risks a self-fulfilling prophecy (an employee treated as a flight risk may disengage as a result of that treatment) and risks decisions being shaped by personal characteristics rather than the actionable factors (overtime, pay, satisfaction) the model also identifies. Aggregated, department-level trends are a safer basis for HR action than individual-level scores.

**Legal and Societal Considerations**
Using personal characteristics such as age, marital status, or distance from home as part of an employment-related risk score would raise discrimination and employment-law concerns in most real jurisdictions. This project treats those columns as data to observe and disclose, not as inputs for any recommended action.

## Key Findings

Overall attrition sits at 16.1%, but varies by department (13.8% to 20.6%) and much more sharply by job role (2.5% to 40%), directly answering Business Requirement 1. All four hypotheses were supported: overtime, income, job satisfaction, and department are each individually associated with attrition, with overtime showing by far the strongest effect and department the weakest. The Modelling notebook's feature importance ranking shows MonthlyIncome, Age, and tenure-related features carrying the most combined predictive weight, overtime is still meaningful there but ranks lower once every other feature is considered together, a useful nuance since it was the strongest single-factor result in the EDA.

## Prevention Measures / Retention Strategy Recommendations

1. **Tackle overtime first**, the strongest single-factor signal in the data, and workload/staffing review in overtime-heavy teams is the most actionable lever available.
2. **Review compensation for lower earners**, MonthlyIncome is both a statistically confirmed driver and the top combined predictor in the model.
3. **Focus retention efforts on Sales and HR**, and specifically Sales Representative and Laboratory Technician roles, the two highest-attrition job roles found in the department/role breakdown.
4. **Improve job satisfaction support** for employees reporting the lowest ratings.

Age, marital status, and distance from home are deliberately excluded from these recommendations, see Ethical Considerations above.

## Model

Logistic Regression and Random Forest were compared using the same preprocessing pipeline and full feature set.
Logistic Regression achieved the strongest overall performance, with an accuracy of 86.1% and a recall of 34.0% for the Attrition = Yes class. Random Forest achieved 85.0% accuracy and 15.0% recall, while the majority-class baseline achieved 83.8% accuracy but 0% recall.
The results demonstrate why accuracy alone is not sufficient for this imbalanced classification problem. Although the majority-class baseline produces relatively high accuracy, it fails to identify any employees who leave.
Logistic Regression was therefore selected as the preferred model because it achieved both the highest overall accuracy and the strongest recall for employees who actually left.
The recall of 34% is still relatively modest, so the model should be treated as a decision-support tool rather than a reliable individual-level prediction system.

## Dashboard Design
An interactive three-page Power BI dashboard was developed to communicate the findings to HR stakeholders and translate the analytical results into actionable insights.

### 1. Employee Attrition Overview

The first page provides a high-level view of employee attrition.

Key KPIs include:

- Total Employees: 1,470
- Employees Stayed: 1,233
- Employees Left: 237
- Overall Attrition Rate: 16.1%

The page also compares attrition across departments, job roles, overtime status and monthly income, with interactive slicers allowing users to explore different employee groups.

### 2. Attrition Drivers

The second page focuses on the main factors associated with employee attrition.

The dashboard explores:

- Monthly Income
- Work-Life Balance
- Years Since Last Promotion
- Overtime
- Job Satisfaction
- Department

The page combines findings from the EDA and hypothesis testing with practical HR recommendations. Overtime is particularly notable, with an attrition rate of 30.5% among employees working overtime compared with 10.4% among employees who do not.

### 3. Predictive Insights

The third page presents the machine learning results.

It includes:

- Top 10 features used by the Random Forest model
- Logistic Regression vs Random Forest performance comparison
- Majority-class baseline comparison
- Best Model: Logistic Regression
- Predictive insights and HR recommendations

Logistic Regression was selected as the preferred model, achieving 86.1% accuracy and 34.0% recall for employees who left.

The dashboard is intended to support exploratory HR decision-making and should not be used to make automated employment decisions.

### Live Power BI Dashboard

https://app.powerbi.com/view?r=eyJrIjoiNDAyOWE4NmUtOWQ1YS00MjM3LThiZGQtZWFiMmI5ZTY3N2Y3IiwidCI6ImMyMzNjMDcyLTEzNWItNDMxZC1hZjU5LTM1ZTA1YmFiZjk0MSIsImMiOjh9


## Limitations

The dataset is synthetic (IBM's sample data), not real employee records, so findings should be treated as illustrative rather than directly actionable for a real organisation. The dataset is relatively small (1,470 rows) and moderately imbalanced (16% attrition), which limits how precisely either model can identify at-risk employees. Two of the four variables tested (MonthlyIncome and JobSatisfaction) failed normality testing, the t-test was used regardless, consistent with the approach taken in the fraud detection project, and the result held up across multiple alternative tests.

## Development Roadmap

With more time I'd want to test additional models, apply resampling techniques such as SMOTE to address the class imbalance more directly rather than working around it with recall, and extend the job-role breakdown into its own formally tested hypothesis rather than a descriptive finding.

## Bugs and Fixes
During development, several issues were encountered and resolved:

- Plotly visualisations rendered correctly in VS Code but were not displayed reliably in GitHub notebook previews. These charts were replaced with Matplotlib and Seaborn visualisations to ensure compatibility.
- The notebook working directory initially caused relative file paths to fail. The project directory structure was corrected so notebooks could consistently access files under `data/clean/`.
- Virtual environment files were accidentally tracked by Git and were removed from version control and added to `.gitignore`.
- Power BI output tables were checked against the machine learning notebook to ensure that model accuracy and recall values were reported consistently.

## Reflection
This project demonstrated the importance of combining exploratory analysis, statistical testing, machine learning and business-focused visualisation rather than relying on a single analytical method.

One of the most important lessons was that accuracy can be misleading when working with an imbalanced target. The majority-class baseline achieved relatively high accuracy while failing to identify any employees who left. This made recall particularly important when comparing predictive models.

The project also highlighted the difference between statistical associations and predictive feature importance. Overtime showed the strongest individual association with attrition, while MonthlyIncome, Age and tenure-related variables ranked highly in the Random Forest feature importance analysis.

Working as a team also required clear ownership of ETL, EDA, statistical analysis, modelling, visualisation and documentation while keeping outputs consistent across notebooks and the final dashboard.

## AI Assistance

AI was used mainly for the machine learning section, to help understand train/test splitting, how to compare models properly, and how to interpret precision/recall trade-offs on an imbalanced target. It was also used as a guide when choosing which statistical test fit which hypothesis. I checked the actual output at each step rather than taking any of it as given.

## Main Data Analysis Libraries

pandas, numpy, matplotlib, seaborn, scipy, pingouin, scikit-learn

## Credits & Acknowledgements
- Code Institute Data Analytics with AI course materials and project template.
- Project team members for contributions across ETL, EDA, statistical testing, machine learning, visualisation and documentation.


