# SQl-Projects
# Hospital Analytics
CREATE DATABASE Healthcare_Hospital_Analytics_System;
USE Healthcare_Hospital_Analytics_System;


-- =========================================================
-- PART 1: TABLE CREATION
-- =========================================================

CREATE TABLE patients (
    patient_id INT PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    age INT,
    gender VARCHAR(50),
    blood_type VARCHAR(20),
    city VARCHAR(100),
    phone VARCHAR(20),
    insurance_provider VARCHAR(100),
    registration_date DATE
);

CREATE TABLE doctors (
    doctor_id INT PRIMARY KEY,
    doctor_name VARCHAR(100),
    specialization VARCHAR(100),
    department VARCHAR(100),
    city VARCHAR(100),
    salary DECIMAL(12,2),
    joining_date DATE,
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES doctors(doctor_id)
);

CREATE TABLE hospitals (
    hospital_id INT PRIMARY KEY,
    hospital_name VARCHAR(150),
    city VARCHAR(100),
    hospital_type VARCHAR(100),
    rating DECIMAL(3,1)
);

CREATE TABLE admissions (
    admission_id INT PRIMARY KEY,
    patient_id INT,
    doctor_id INT,
    hospital_id INT,
    admission_date DATE,
    discharge_date DATE,
    admission_type VARCHAR(50),
    room_number INT,
    billing_amount DECIMAL(12,2),
    test_result VARCHAR(100),
    medical_condition VARCHAR(200),
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id),
    FOREIGN KEY (hospital_id) REFERENCES hospitals(hospital_id)
);

CREATE TABLE medicines (
    medicine_id INT PRIMARY KEY,
    medicine_name VARCHAR(150),
    category VARCHAR(100),
    manufacturer VARCHAR(150),
    price DECIMAL(10,2),
    stock INT
);

CREATE TABLE prescriptions (
    prescription_id INT PRIMARY KEY,
    patient_id INT,
    doctor_id INT,
    medicine_id INT,
    dosage VARCHAR(100),
    prescription_date DATE,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id),
    FOREIGN KEY (medicine_id) REFERENCES medicines(medicine_id)
);

CREATE TABLE payments (
    payment_id INT PRIMARY KEY,
    admission_id INT,
    payment_date DATE,
    amount DECIMAL(12,2),
    payment_method VARCHAR(50),
    payment_status VARCHAR(50),
    FOREIGN KEY (admission_id) REFERENCES admissions(admission_id)
);


-- =========================================================
-- PART 2: DATA EXPLORATION
-- =========================================================

SELECT COUNT(*) AS total_patients
FROM patients;

SELECT DISTINCT blood_type
FROM patients;

SELECT DISTINCT city
FROM patients;

SELECT gender, COUNT(*) AS total_patients
FROM patients
GROUP BY gender;

SELECT
    CASE
        WHEN age < 30 THEN 'Young'
        WHEN age BETWEEN 30 AND 50 THEN 'Middle'
        ELSE 'Senior'
    END AS age_group,
    COUNT(*) AS total_patients
FROM patients
GROUP BY age_group;

SELECT
    MAX(billing_amount) AS highest_billing,
    MIN(billing_amount) AS lowest_billing,
    ROUND(AVG(billing_amount),2) AS average_billing
FROM admissions;

SELECT *
FROM admissions
WHERE billing_amount IS NULL;

SELECT
    patient_id,
    COUNT(*) AS duplicate_count
FROM patients
GROUP BY patient_id
HAVING COUNT(*) > 1;

SELECT
    CONCAT(first_name, ' ', last_name) AS full_name,
    UPPER(CONCAT(first_name, ' ', last_name)) AS uppercase_name,
    LOWER(CONCAT(first_name, ' ', last_name)) AS lowercase_name,
    LENGTH(CONCAT(first_name, ' ', last_name)) AS name_length
FROM patients;

SELECT
    registration_date,
    YEAR(registration_date) AS year,
    MONTH(registration_date) AS month,
    DAY(registration_date) AS day
FROM patients;

SELECT
    first_name,
    registration_date,
    DATEDIFF(CURDATE(), registration_date) AS days_since_registration,
    CASE
        WHEN registration_date < CURDATE() THEN 'Past'
        WHEN registration_date = CURDATE() THEN 'Today'
        ELSE 'Future'
    END AS status
FROM patients;


-- =========================================================
-- PART 3: PATIENT ANALYSIS
-- =========================================================

SELECT
    CASE
        WHEN age < 30 THEN 'Young'
        WHEN age BETWEEN 30 AND 50 THEN 'Middle'
        ELSE 'Senior'
    END AS age_group,
    COUNT(*) AS total_patients
FROM patients
GROUP BY age_group;

SELECT gender, COUNT(*) AS total_patients
FROM patients
GROUP BY gender;

SELECT city, COUNT(*) AS total_patients
FROM patients
GROUP BY city;

SELECT insurance_provider, COUNT(*) AS total_patients
FROM patients
GROUP BY insurance_provider;

SELECT medical_condition, COUNT(*) AS total_patients
FROM admissions
GROUP BY medical_condition;

SELECT
    CONCAT(p.first_name, ' ', p.last_name) AS patient_name,
    a.billing_amount
FROM patients p
JOIN admissions a
    ON p.patient_id = a.patient_id
ORDER BY a.billing_amount DESC
LIMIT 1;

SELECT
    CONCAT(p.first_name, ' ', p.last_name) AS patient_name,
    a.billing_amount
FROM patients p
JOIN admissions a
    ON p.patient_id = a.patient_id
WHERE a.billing_amount > (
    SELECT AVG(billing_amount)
    FROM admissions
);

SELECT
    admission_id,
    patient_id,
    billing_amount
FROM admissions
ORDER BY billing_amount DESC
LIMIT 10;

SELECT *
FROM admissions
WHERE billing_amount BETWEEN 30000 AND 80000;

SELECT *
FROM patients
WHERE city IN ('Indore', 'Bhopal');

SELECT *
FROM patients
WHERE city NOT IN ('Indore', 'Bhopal');

SELECT *
FROM patients
WHERE first_name LIKE 'A%';

SELECT *
FROM patients
WHERE first_name LIKE '%a%';

SELECT *
FROM admissions
WHERE medical_condition IN ('Diabetes', 'Heart Disease');

SELECT
    patient_id,
    COALESCE(phone, 'Not Available') AS phone
FROM patients;

SELECT
    patient_id,
    NULLIF(city, '') AS city
FROM patients;

SELECT
    patient_id,
    CAST(age AS DECIMAL(5,2)) AS converted_age
FROM patients;

SELECT
    admission_id,
    ROUND(billing_amount, 2) AS rounded_bill
FROM admissions;


-- =========================================================
-- PART 4: HOSPITAL PERFORMANCE
-- =========================================================

SELECT
    h.hospital_id,
    h.hospital_name,
    COUNT(DISTINCT a.patient_id) AS total_patients,
    COUNT(a.admission_id) AS total_admissions,
    SUM(a.billing_amount) AS total_revenue,
    ROUND(AVG(a.billing_amount),2) AS average_bill,
    MIN(a.billing_amount) AS minimum_bill,
    MAX(a.billing_amount) AS maximum_bill,
    ROUND(AVG(DATEDIFF(a.discharge_date, a.admission_date)),2) AS average_stay,
    SUM(CASE WHEN a.admission_type = 'Emergency' THEN 1 ELSE 0 END) AS emergency_admissions,
    SUM(CASE WHEN a.admission_type <> 'Emergency' THEN 1 ELSE 0 END) AS normal_admissions
FROM hospitals h
LEFT JOIN admissions a
    ON h.hospital_id = a.hospital_id
GROUP BY h.hospital_id, h.hospital_name;

SELECT
    h.hospital_name,
    SUM(a.billing_amount) AS total_revenue
FROM hospitals h
JOIN admissions a
    ON h.hospital_id = a.hospital_id
GROUP BY h.hospital_id, h.hospital_name
HAVING SUM(a.billing_amount) > 100000;


-- =========================================================
-- PART 5: DOCTOR PERFORMANCE
-- =========================================================

SELECT
    d.doctor_name,
    d.department,
    COUNT(a.patient_id) AS patients_treated,
    SUM(a.billing_amount) AS total_revenue,
    ROUND(AVG(a.billing_amount),2) AS average_billing,
    MAX(a.billing_amount) AS highest_billing,
    MIN(a.billing_amount) AS lowest_billing
FROM doctors d
LEFT JOIN admissions a
    ON d.doctor_id = a.doctor_id
GROUP BY d.doctor_id, d.doctor_name, d.department;

SELECT
    d.doctor_name,
    SUM(a.billing_amount) AS total_revenue,
    RANK() OVER (
        ORDER BY SUM(a.billing_amount) DESC
    ) AS revenue_rank
FROM doctors d
JOIN admissions a
    ON d.doctor_id = a.doctor_id
GROUP BY d.doctor_id, d.doctor_name;

SELECT
    d.doctor_name,
    a.admission_id,
    a.billing_amount,
    LAG(a.billing_amount) OVER (
        PARTITION BY d.doctor_id
        ORDER BY a.admission_date
    ) AS previous_billing,
    LEAD(a.billing_amount) OVER (
        PARTITION BY d.doctor_id
        ORDER BY a.admission_date
    ) AS next_billing
FROM doctors d
JOIN admissions a
    ON d.doctor_id = a.doctor_id;


-- =========================================================
-- PART 6: MULTIPLE TABLE JOIN
-- =========================================================

SELECT
    CONCAT(p.first_name, ' ', p.last_name) AS patient,
    d.doctor_name,
    d.specialization,
    h.hospital_name,
    h.city,
    a.admission_date,
    a.discharge_date,
    a.admission_type,
    a.medical_condition,
    a.billing_amount,
    py.payment_status
FROM patients p
JOIN admissions a
    ON p.patient_id = a.patient_id
JOIN doctors d
    ON a.doctor_id = d.doctor_id
JOIN hospitals h
    ON a.hospital_id = h.hospital_id
LEFT JOIN payments py
    ON a.admission_id = py.admission_id;


-- =========================================================
-- PART 7: DIFFERENT JOINS
-- =========================================================

SELECT *
FROM patients p
INNER JOIN admissions a
    ON p.patient_id = a.patient_id;

SELECT *
FROM patients p
LEFT JOIN admissions a
    ON p.patient_id = a.patient_id;

SELECT *
FROM admissions a
RIGHT JOIN patients p
    ON a.patient_id = p.patient_id;

SELECT
    p.first_name,
    h.hospital_name
FROM patients p
CROSS JOIN hospitals h;

SELECT
    d1.doctor_name AS doctor,
    d2.doctor_name AS manager
FROM doctors d1
LEFT JOIN doctors d2
    ON d1.manager_id = d2.doctor_id;

-- MySQL FULL JOIN alternative
SELECT p.patient_id, p.first_name, a.admission_id
FROM patients p
LEFT JOIN admissions a
    ON p.patient_id = a.patient_id

UNION

SELECT p.patient_id, p.first_name, a.admission_id
FROM patients p
RIGHT JOIN admissions a
    ON p.patient_id = a.patient_id;


-- =========================================================
-- PART 8: SUBQUERIES
-- =========================================================

SELECT *
FROM admissions
WHERE billing_amount > (
    SELECT AVG(billing_amount)
    FROM admissions
);

SELECT
    d.doctor_name,
    SUM(a.billing_amount) AS revenue
FROM doctors d
JOIN admissions a
    ON d.doctor_id = a.doctor_id
GROUP BY d.doctor_id, d.doctor_name
HAVING SUM(a.billing_amount) > (
    SELECT AVG(total_revenue)
    FROM (
        SELECT SUM(billing_amount) AS total_revenue
        FROM admissions
        GROUP BY hospital_id
    ) x
);

SELECT
    h.hospital_name,
    SUM(a.billing_amount) AS total_revenue
FROM hospitals h
JOIN admissions a
    ON h.hospital_id = a.hospital_id
GROUP BY h.hospital_id, h.hospital_name
ORDER BY total_revenue DESC
LIMIT 1;

SELECT
    h.city,
    AVG(a.billing_amount) AS average_billing
FROM hospitals h
JOIN admissions a
    ON h.hospital_id = a.hospital_id
GROUP BY h.city
ORDER BY average_billing DESC
LIMIT 1;

SELECT *
FROM patients p
WHERE EXISTS (
    SELECT 1
    FROM admissions a
    WHERE a.patient_id = p.patient_id
);

SELECT *
FROM patients p
WHERE NOT EXISTS (
    SELECT 1
    FROM admissions a
    WHERE a.patient_id = p.patient_id
);

SELECT *
FROM doctors
WHERE salary > ANY (
    SELECT salary
    FROM doctors
    WHERE department = 'Cardiology'
);

SELECT *
FROM doctors
WHERE salary > ALL (
    SELECT salary
    FROM doctors
    WHERE department = 'Cardiology'
);


-- =========================================================
-- PART 9: CTE
-- =========================================================

WITH hospital_revenue AS (
    SELECT
        hospital_id,
        SUM(billing_amount) AS total_revenue
    FROM admissions
    GROUP BY hospital_id
),
hospital_ranking AS (
    SELECT
        hospital_id,
        total_revenue,
        RANK() OVER (
            ORDER BY total_revenue DESC
        ) AS hospital_rank
    FROM hospital_revenue
)
SELECT
    h.hospital_name,
    hr.total_revenue,
    hr.hospital_rank
FROM hospital_ranking hr
JOIN hospitals h
    ON hr.hospital_id = h.hospital_id
WHERE hr.hospital_rank <= 3;


WITH patient_admission AS (
    SELECT
        patient_id,
        admission_id,
        hospital_id,
        billing_amount
    FROM admissions
),
hospital_revenue AS (
    SELECT
        hospital_id,
        SUM(billing_amount) AS total_revenue
    FROM patient_admission
    GROUP BY hospital_id
)
SELECT
    h.hospital_name,
    hr.total_revenue
FROM hospital_revenue hr
JOIN hospitals h
    ON hr.hospital_id = h.hospital_id;


-- =========================================================
-- PART 10: RECURSIVE CTE - DOCTOR HIERARCHY
-- =========================================================

WITH RECURSIVE doctor_hierarchy AS (
    SELECT
        doctor_id,
        doctor_name,
        manager_id,
        1 AS level
    FROM doctors
    WHERE manager_id IS NULL

    UNION ALL

    SELECT
        d.doctor_id,
        d.doctor_name,
        d.manager_id,
        dh.level + 1
    FROM doctors d
    JOIN doctor_hierarchy dh
        ON d.manager_id = dh.doctor_id
)
SELECT *
FROM doctor_hierarchy
ORDER BY level, doctor_id;


-- =========================================================
-- PART 11: WINDOW FUNCTIONS
-- =========================================================

SELECT
    admission_date,
    billing_amount,
    SUM(billing_amount) OVER (
        ORDER BY admission_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM admissions;

SELECT
    admission_date,
    billing_amount,
    AVG(billing_amount) OVER (
        ORDER BY admission_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_average
FROM admissions;

SELECT
    h.hospital_name,
    a.admission_id,
    a.billing_amount,
    RANK() OVER (
        PARTITION BY h.hospital_id
        ORDER BY a.billing_amount DESC
    ) AS hospital_rank
FROM hospitals h
JOIN admissions a
    ON h.hospital_id = a.hospital_id;

SELECT *
FROM (
    SELECT
        h.hospital_name,
        a.admission_id,
        a.billing_amount,
        ROW_NUMBER() OVER (
            PARTITION BY h.hospital_id
            ORDER BY a.billing_amount DESC
        ) AS rn
    FROM hospitals h
    JOIN admissions a
        ON h.hospital_id = a.hospital_id
) x
WHERE rn <= 3;

SELECT
    admission_id,
    billing_amount,
    FIRST_VALUE(billing_amount) OVER (
        ORDER BY admission_date
    ) AS first_billing,
    LAST_VALUE(billing_amount) OVER (
        ORDER BY admission_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_billing
FROM admissions;

SELECT
    admission_id,
    billing_amount,
    NTH_VALUE(billing_amount, 2) OVER (
        ORDER BY admission_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_billing
FROM admissions;

SELECT
    admission_id,
    billing_amount,
    SUM(billing_amount) OVER () AS total_billing,
    AVG(billing_amount) OVER () AS average_billing,
    COUNT(*) OVER () AS total_records,
    MIN(billing_amount) OVER () AS minimum_billing,
    MAX(billing_amount) OVER () AS maximum_billing
FROM admissions;

SELECT
    admission_id,
    billing_amount,
    PERCENT_RANK() OVER (
        ORDER BY billing_amount
    ) AS percentage_rank
FROM admissions;

SELECT
    admission_id,
    billing_amount,
    CUME_DIST() OVER (
        ORDER BY billing_amount
    ) AS cumulative_distribution
FROM admissions;

SELECT
    admission_id,
    billing_amount,
    NTILE(4) OVER (
        ORDER BY billing_amount
    ) AS quartile_group
FROM admissions;


-- =========================================================
-- PART 12: HOSPITAL REVENUE DASHBOARD
-- =========================================================

WITH hospital_data AS (
    SELECT
        h.hospital_id,
        h.hospital_name,
        h.city,
        COUNT(DISTINCT a.patient_id) AS total_patients,
        COUNT(a.admission_id) AS total_admissions,
        SUM(
            CASE
                WHEN a.admission_type = 'Emergency' THEN 1
                ELSE 0
            END
        ) AS emergency_cases,
        SUM(
            CASE
                WHEN a.admission_type <> 'Emergency' THEN 1
                ELSE 0
            END
        ) AS normal_cases,
        SUM(a.billing_amount) AS total_revenue,
        AVG(a.billing_amount) AS average_revenue,
        MAX(a.billing_amount) AS highest_bill,
        MIN(a.billing_amount) AS lowest_bill
    FROM hospitals h
    LEFT JOIN admissions a
        ON h.hospital_id = a.hospital_id
    GROUP BY h.hospital_id, h.hospital_name, h.city
)
SELECT
    *,
    RANK() OVER (
        ORDER BY total_revenue DESC
    ) AS hospital_rank,
    ROUND(
        total_revenue /
        SUM(total_revenue) OVER () * 100,
        2
    ) AS revenue_percentage
FROM hospital_data;


-- =========================================================
-- PART 13: INSERT / UPDATE / DELETE
-- =========================================================

INSERT INTO patients
VALUES (
    11,
    'Arjun',
    'Sharma',
    28,
    'Male',
    'A+',
    'Indore',
    '9876543210',
    'LIC',
    '2026-05-01'
);

UPDATE patients
SET city = 'Bhopal'
WHERE patient_id = 11;

DELETE FROM patients
WHERE patient_id = 11;

INSERT INTO medicines
VALUES (
    509,
    'Azithromycin',
    'Antibiotic',
    'Cipla',
    120,
    300
);

UPDATE medicines
SET price = 130,
    stock = stock + 50
WHERE medicine_id = 509;

-- MySQL UPSERT
INSERT INTO medicines
(
    medicine_id,
    medicine_name,
    category,
    manufacturer,
    price,
    stock
)
VALUES
(
    509,
    'Azithromycin',
    'Antibiotic',
    'Cipla',
    120,
    300
)
ON DUPLICATE KEY UPDATE
    price = VALUES(price),
    stock = VALUES(stock);


-- =========================================================
-- PART 14: VIEWS
-- =========================================================

CREATE VIEW high_value_patients AS
SELECT
    p.patient_id,
    CONCAT(p.first_name, ' ', p.last_name) AS patient_name,
    a.billing_amount
FROM patients p
JOIN admissions a
    ON p.patient_id = a.patient_id
WHERE a.billing_amount > (
    SELECT AVG(billing_amount)
    FROM admissions
);

CREATE VIEW hospital_performance AS
SELECT
    h.hospital_name,
    COUNT(a.admission_id) AS total_admissions,
    SUM(a.billing_amount) AS total_revenue,
    AVG(a.billing_amount) AS average_bill
FROM hospitals h
LEFT JOIN admissions a
    ON h.hospital_id = a.hospital_id
GROUP BY h.hospital_id, h.hospital_name;

CREATE VIEW doctor_performance AS
SELECT
    d.doctor_name,
    d.department,
    COUNT(a.admission_id) AS patients_treated,
    SUM(a.billing_amount) AS total_revenue,
    AVG(a.billing_amount) AS average_billing
FROM doctors d
LEFT JOIN admissions a
    ON d.doctor_id = a.doctor_id
GROUP BY d.doctor_id, d.doctor_name, d.department;

SELECT *
FROM high_value_patients;

SELECT *
FROM hospital_performance;

SELECT *
FROM doctor_performance;


-- =========================================================
-- PART 15: CREATE ANALYTICAL TABLE
-- =========================================================

CREATE TABLE hospital_revenue_summary AS
SELECT
    h.hospital_id,
    h.hospital_name,
    h.city,
    COUNT(a.admission_id) AS total_admissions,
    SUM(a.billing_amount) AS total_revenue,
    AVG(a.billing_amount) AS average_bill
FROM hospitals h
LEFT JOIN admissions a
    ON h.hospital_id = a.hospital_id
GROUP BY
    h.hospital_id,
    h.hospital_name,
    h.city;


-- =========================================================
-- PART 16: SET OPERATIONS
-- =========================================================

-- UNION
SELECT city
FROM patients
WHERE city = 'Indore'

UNION

SELECT city
FROM patients
WHERE city = 'Bhopal';


-- UNION ALL
SELECT city
FROM patients
WHERE city = 'Indore'

UNION ALL

SELECT city
FROM patients
WHERE city = 'Bhopal';


-- INTERSECT alternative for MySQL
SELECT DISTINCT p1.city
FROM patients p1
INNER JOIN patients p2
    ON p1.city = p2.city
WHERE p1.city = 'Indore'
AND p2.city = 'Bhopal';


-- EXCEPT alternative using NOT EXISTS
SELECT DISTINCT p.city
FROM patients p
WHERE p.city = 'Indore'
AND NOT EXISTS (
    SELECT 1
    FROM patients p2
    WHERE p2.city = 'Bhopal'
    AND p2.city = p.city
);


-- =========================================================
-- PART 17: PIVOT USING CASE
-- =========================================================

SELECT
    h.hospital_name,
    SUM(
        CASE
            WHEN a.admission_type = 'Emergency' THEN 1
            ELSE 0
        END
    ) AS emergency,
    SUM(
        CASE
            WHEN a.admission_type = 'Regular' THEN 1
            ELSE 0
        END
    ) AS regular
FROM hospitals h
LEFT JOIN admissions a
    ON h.hospital_id = a.hospital_id
GROUP BY h.hospital_id, h.hospital_name;


-- UNPIVOT alternative
SELECT hospital_name, 'Emergency' AS admission_type, emergency AS total
FROM (
    SELECT
        h.hospital_name,
        SUM(CASE WHEN a.admission_type = 'Emergency' THEN 1 ELSE 0 END) AS emergency,
        SUM(CASE WHEN a.admission_type = 'Regular' THEN 1 ELSE 0 END) AS regular
    FROM hospitals h
    LEFT JOIN admissions a
        ON h.hospital_id = a.hospital_id
    GROUP BY h.hospital_id, h.hospital_name
) x

UNION ALL

SELECT hospital_name, 'Regular', regular
FROM (
    SELECT
        h.hospital_name,
        SUM(CASE WHEN a.admission_type = 'Emergency' THEN 1 ELSE 0 END) AS emergency,
        SUM(CASE WHEN a.admission_type = 'Regular' THEN 1 ELSE 0 END) AS regular
    FROM hospitals h
    LEFT JOIN admissions a
        ON h.hospital_id = a.hospital_id
    GROUP BY h.hospital_id, h.hospital_name
) x;


-- =========================================================
-- PART 18: JSON DATA
-- =========================================================

ALTER TABLE patients
ADD COLUMN additional_info JSON;

UPDATE patients
SET additional_info = JSON_OBJECT(
    'allergy', 'Penicillin',
    'emergency_contact', 'Father',
    'language', 'Hindi'
)
WHERE patient_id = 1;

SELECT
    patient_id,
    JSON_EXTRACT(additional_info, '$.allergy') AS allergy,
    JSON_EXTRACT(additional_info, '$.emergency_contact') AS emergency_contact,
    JSON_EXTRACT(additional_info, '$.language') AS language
FROM patients
WHERE additional_info IS NOT NULL;

SELECT
    patient_id,
    additional_info ->> '$.allergy' AS allergy,
    additional_info ->> '$.emergency_contact' AS emergency_contact,
    additional_info ->> '$.language' AS language
FROM patients
WHERE additional_info IS NOT NULL;


-- =========================================================
-- PART 19: QUERY OPTIMIZATION
-- =========================================================

EXPLAIN
SELECT
    p.first_name,
    p.last_name,
    a.billing_amount
FROM patients p
JOIN admissions a
    ON p.patient_id = a.patient_id
WHERE a.billing_amount > 50000;

EXPLAIN
SELECT
    h.hospital_name,
    SUM(a.billing_amount) AS total_revenue
FROM hospitals h
JOIN admissions a
    ON h.hospital_id = a.hospital_id
GROUP BY h.hospital_id, h.hospital_name;


-- =========================================================
-- PART 20: INDEXING FOR OPTIMIZATION
-- =========================================================

CREATE INDEX idx_patient_city
ON patients(city);

CREATE INDEX idx_admission_patient
ON admissions(patient_id);

CREATE INDEX idx_admission_doctor
ON admissions(doctor_id);

CREATE INDEX idx_admission_hospital
ON admissions(hospital_id);

CREATE INDEX idx_billing
ON admissions(billing_amount);

CREATE INDEX idx_admission_date
ON admissions(admission_date);

EXPLAIN
SELECT *
FROM admissions
WHERE billing_amount > 50000;
____________________________________________________________________________________________________________________________________________________________________________________________________________________________
