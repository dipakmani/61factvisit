import pandas as pd
import numpy as np
import random
from faker import Faker

fake = Faker()
np.random.seed(42)
random.seed(42)

# =========================
# PARAMETERS
# =========================
TOTAL_RECORDS = 500000
TOTAL_PATIENTS = 100000       # 20% will revisit
REPEAT_PATIENTS = 20000
TOTAL_DOCTORS = 500
TOTAL_DEPARTMENTS = 50
TOTAL_HOSPITALS = 30
TOTAL_DIAGNOSIS = 200
TOTAL_INSURANCE = 50

# =========================
# DIMENSION DATA
# =========================
# Patients
patients = []
for pid in range(1, TOTAL_PATIENTS+1):
    gender = "Male" if np.random.rand() < 0.6 else "Female"
    dob = fake.date_of_birth(minimum_age=1, maximum_age=90)
    age = (pd.Timestamp("2025-01-01") - pd.to_datetime(dob)).days // 365
    patients.append({
        "PatientID": pid,
        "Patient_Fullname": fake.name_male() if gender=="Male" else fake.name_female(),
        "Patient_Gender": gender,
        "Patient_age": age,
        "patientdob": dob,
        "patientaddress": fake.street_address(),
        "patientcity": fake.city(),
        "patientstate": fake.state(),
        "patientcountry": fake.country(),
        "patientpostalcode": fake.postcode(),
        "patient_satisfactionscore": random.randint(1,10),
        "patient_bloodgroup": random.choice(["A+","A-","B+","B-","O+","O-","AB+","AB-"])
    })
patients_df = pd.DataFrame(patients)

# Doctors
doctors = []
for did in range(1, TOTAL_DOCTORS+1):
    doctors.append({
        "DoctorID": did,
        "DoctorFullName": fake.name(),
        "doctor_specialization": random.choice(["Cardiology","Neurology","Orthopedics","Pediatrics","Oncology","Dermatology"]),
        "doctoryears_experience": random.randint(1,40),
        "doctoremail": fake.email(),
        "doctor_shift_type": random.choice(["Day","Night","Rotational"])
    })
doctors_df = pd.DataFrame(doctors)

# Departments
departments = []
for deptid in range(1, TOTAL_DEPARTMENTS+1):
    departments.append({
        "departmentid": deptid,
        "departmentname": random.choice(["Cardiology","Neurology","Orthopedics","Pediatrics","Oncology","Dermatology"]),
        "department_speciality": fake.word(),
        "Departmentreferral": fake.word(),
        "floor_number": random.randint(1,10)
    })
departments_df = pd.DataFrame(departments)

# Hospitals
hospitals = []
for hid in range(1, TOTAL_HOSPITALS+1):
    hospitals.append({
        "hospitalid": hid,
        "hospital_name": fake.company(),
        "hospital_provideer_name": fake.company_suffix(),
        "hospital_type": random.choice(["Private","Government","Charity"]),
        "bed_capacity": random.randint(50,1000),
        "hospital_address": fake.address(),
        "hospitalstate": fake.state(),
        "hospitalcity": fake.city(),
        "hospitalcountry": fake.country(),
        "hospitalpostalcode": fake.postcode()
    })
hospitals_df = pd.DataFrame(hospitals)

# Diagnosis
diagnosis = []
for did in range(1, TOTAL_DIAGNOSIS+1):
    diagnosis.append({
        "diagnosisid": did,
        "diagnosiscode": f"D{did:04d}",
        "diagnosisdiscription": fake.sentence(),
        "diagnosiscategory": random.choice(["Infectious","Chronic","Acute","Genetic","Lifestyle"])
    })
diagnosis_df = pd.DataFrame(diagnosis)

# Insurance
insurance = []
for iid in range(1, TOTAL_INSURANCE+1):
    insurance.append({
        "insuranceproviderid": iid,
        "insuranceprovidername": fake.company(),
        "insuranceplan_type": random.choice(["Basic","Premium","Gold","Platinum"]),
        "coverage_percentage": random.choice([50,60,70,80,90,100]),
        "insurance_email": fake.company_email()
    })
insurance_df = pd.DataFrame(insurance)

# =========================
# FACT TABLE (VISITS)
# =========================
fact_rows = []

for vid in range(1, TOTAL_RECORDS+1):
    patient = patients_df.sample(1).iloc[0]
    doctor = doctors_df.sample(1).iloc[0]
    dept = departments_df.sample(1).iloc[0]
    hosp = hospitals_df.sample(1).iloc[0]
    diag = diagnosis_df.sample(1).iloc[0]
    ins = insurance_df.sample(1).iloc[0]

    admission_date = fake.date_between(start_date="-3y", end_date="today")
    discharge_date = admission_date + pd.Timedelta(days=random.randint(1,15))

    total_billing = random.randint(5000,100000)
    insurance_cover = int(total_billing * ins.coverage_percentage / 100)
    patient_cover = total_billing - insurance_cover

    fact_rows.append({
        "VisitID": vid,
        "Admission_Date": admission_date,
        "Discharge_Date": discharge_date,
        **patient.to_dict(),
        **doctor.to_dict(),
        **dept.to_dict(),
        **hosp.to_dict(),
        "total_billing_amount": total_billing,
        "insurancecoveredamount": insurance_cover,
        "patient_covered_amount": patient_cover,
        "full_date": admission_date,
        **diag.to_dict(),
        **ins.to_dict()
    })

fact_df = pd.DataFrame(fact_rows)

# =========================
# SAVE TO CSV
# =========================
fact_df.to_csv("Fact_Visits.csv", index=False)
print("✅ Fact_Visits.csv generated with", len(fact_df), "records")
