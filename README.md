# Healthcare Management System – SQL-Based Data Analysis
![HealthCare](https://raw.githubusercontent.com/adnanalam8360/HealthCare-Management-System/refs/heads/main/Healthcare-img.png)

## Project Overview:
This project focuses on building a Healthcare Management System using structured data analysis with SQL. It involves four core entities: Doctors, Appointments, Medical Procedures, and Billing. The objective is to extract meaningful insights from patient visits, doctor performance, procedures conducted, and financial billing data to support efficient healthcare decision-making.

### Key operations include:

Tracking procedure frequency and identifying repeated treatments.

Estimating revenue per doctor and evaluating average billing per procedure.

Analyzing patient behavior, appointment trends, and time gaps between procedures.

Using SQL JOINs, window functions, and common table expressions (CTEs) to create advanced queries.

The project simulates real-world healthcare data flows and demonstrates how SQL can be used to power data-driven decisions in a clinical environment.

# Steps
I sourced the dataset from Kaggle, which initially contained unstructured and inconsistent entries. To ensure data quality, I performed thorough data cleaning using Microsoft Excel—addressing issues such as missing values, formatting inconsistencies, and duplicates. After preprocessing, I imported the cleaned tables into MySQL Server and adjusted column data types appropriately, including converting date fields to a standardized format. With a clean and well-structured database in place, I proceeded to perform in-depth SQL-based analysis.

# 1.Which doctors have the most patient appointments per month?

select
  d.DoctorName,
  month(ap.Date) as month,
  count(ap.AppointmentID) as appointment_count
from
  appointment ap 
  join doctor d on ap.DoctorID=d.DoctorID
group by 
  1,2
order by 
  appointment_count desc;

# Insight Gained:
Identifies which doctors see the highest number of patients each month.

# Use:
-> Helps in workload distribution.
-> Aids in performance reviews and scheduling.
-> Useful for identifying popular or overburdened doctors.

# 2.Which specialties are most in demand based on appointment count?

select
  d.Specialization,
  count(ap.AppointmentID) as appointment_count
from appointment ap
  join Doctor d on ap.DoctorID=d.DoctorID
group by 1
order by 
  appointment_count desc;

# Insight Gained:
Shows which medical specialties have the highest appointment volume.

# Use:
-> Supports resource allocation decisions (e.g., hiring more specialists).
->Helps in expanding popular departments.
->Indicates patient health trends over time.
# With Time Analysis
select
  d.Specialization AS specialty,
  YEAR(ap.Date) AS year,
  COUNT(ap.AppointmentID) AS appointment_count,
  RANK() OVER (PARTITION BY YEAR(ap.Date) ORDER BY COUNT(ap.AppointmentID) DESC) AS specialty_rank
from appointment ap 
  join Doctor d on ap.DoctorID = d.DoctorID
group by
  d.Specialization,
  YEAR(ap.Date)
order by
  year DESC,
  appointment_count DESC;

# Insight Gained:
Tracks the most in-demand specialties year-over-year.

# Use:
->Identifies trends in patient needs and emerging health concerns.
->Helps in long-term strategic planning.
->Justifies investment in training or technology for certain specialties.

# 3.What is the average patient load per doctor per week?

select
  d.DoctorName,
  count(distinct ap.patientID) as total_patient,
  count(distinct week(ap.Date)) as weeks_worked,
  round(count(distinct ap.patientID)/count(distinct week(ap.Date)),2) as average_patient_per_week
from appointment ap
  join doctor d on ap.DoctorID=d.DoctorID
group by 1
order by 
  average_patient_per_week desc;

# Insight Gained:
Measures how many unique patients a doctor handles weekly on average.

# Use:
-> Prevents doctor burnout by balancing workloads.
-> Optimizes scheduling and appointment allocation.
-> Supports hiring and staffing decisions.

# 4.How many procedures does each doctor perform on average per month?

select
  d.DoctorName,
  count(mp.ProcedureID) as total_procedure,
  count(distinct mp.ProcedureID)/count(distinct month(ap.date)) as average_procedure_per_month
from doctor d 
  join appointment ap on d.DoctorID=ap.DoctorID
  join medical_procedure mp on ap.AppointmentID=mp.AppointmentID
group by 1
order by 
  average_procedure_per_month desc;

# Insight Gained:
Calculates procedure productivity for each doctor.

# Use:
->Aids in evaluating doctor efficiency and performance.
->Helps in training needs assessment.
->Useful for reward/incentive programs.

# 5.What is the average billing amount per procedure or appointment?

select
  mp.ProcedureName,
  count(distinct mp.ProcedureID) as total_procedures,
  sum(b.amount)/count(distinct mp.ProcedureID) as average_billing_amount_per_procedure
from medical_procedure mp 
  join appointment ap on mp.AppointmentID=ap.AppointmentID
  join billing b on ap.PatientID=b.PatientID
group by 1
  order by average_billing_amount_per_procedure desc;

# Insight Gained:
Reveals average revenue generated per procedure.

# Use:
-> Assists in pricing strategies.
-> Helps detect anomalies in billing.
->Useful for revenue forecasting and budgeting.

# 6.Which department generates the highest revenue?

select
  d.Specialization,
  sum(b.amount) as total_revenue
from doctor d 
  join appointment ap on d.DoctorID=ap.DoctorID
  join billing b on ap.PatientID=b.PatientID
group by 1
order by 
  total_revenue
limit 1;

# Insight Gained:
Identifies which specialization contributes the most to overall revenue.

# Use:
-> Prioritizes investment in high-revenue departments.
-> Justifies expansion or tech upgrades for specific areas.
->Informs strategic decisions on service offerings.

# 7.What are the most frequently performed procedures?

select 
  ProcedureName,count(*) as no_procedure_performed
from medical_procedure
group by 1
order by 
  no_procedure_performed desc;

# Insight Gained:
Lists procedures with the highest volume.

# Use:
-> Optimizes inventory management (medical supplies).
-> Guides staff training based on frequently used procedures.
->Indicates common health issues in the population.

# 8.What is the average cost per procedure across departments?

select
  d.Specialization as Department,
  count(mp.ProcedureID) as total_procedure,
  sum(b.amount)/count(mp.ProcedureId) as average_cost_per_prodecure
from doctor d
  join appointment ap on d.DoctorID=ap.DoctorID
  join billing b on ap.PatientID=b.PatientID
  join medical_procedure mp on ap.AppointmentID=mp.AppointmentID
group by 1
order by 
  average_cost_per_prodecure desc;

# Insight Gained:
Compares costs of procedures by department/specialization.

# Use:
-> Identifies cost-efficiency gaps.
-> Guides financial planning and patient billing strategies.
-> Assists in benchmarking departmental performance.

# 9.Are there patients undergoing repeated procedures? If yes, how frequently?

select 
  ap.patientID,
  mp.ProcedureName,
  count(mp.AppointmentId) as ProcedureCount
from appointment ap 
  join medical_procedure mp on ap.AppointmentID=mp.AppointmentID
group by 1,2
having 
  count(mp.AppointmentId)>1
order by 
  ProcedureCount desc;

# Insight Gained:
Highlights patients receiving the same procedure multiple times.

# Use:
-> Detects chronic cases or potential procedural inefficiencies.
-> Improves patient follow-up and care planning.
-> Flags potential issues for quality control.

# 10.What is the total revenue generated by each doctor?

With doctorRevenue as(
select
  d.DoctorName,
  sum(b.Amount) as total_revenue
from doctor d 
  join appointment ap on D.doctorID=ap.DoctorID
  join billing b on ap.PatientID=b.PatientID
group by 1
)
select 
  *
from doctorRevenue
order by 
  total_revenue desc
limit 5;

# Insight Gained:
Ranks doctors by total revenue generated through patient appointments.

# Use:
-> Recognizes top-performing doctors financially.
-> Helps plan performance incentives or bonuses.
-> Useful for financial audits and departmental contributions.

# Conclusion
This Healthcare Management System project demonstrates how structured SQL analysis can transform raw medical data into valuable insights. By leveraging joins, CTEs, and window functions, we uncovered trends in doctor performance, patient behavior, procedure frequency, and revenue distribution. These insights can help hospitals improve resource allocation, optimize scheduling, enhance patient care, and support data-driven decision-making for better healthcare outcomes.
