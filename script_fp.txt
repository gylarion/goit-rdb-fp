CREATE DATABASE pandemic;
USE pandemic;

SELECT * 
FROM infectious_cases
LIMIT 10;

CREATE TABLE entities (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity VARCHAR(255) NOT NULL,
    code VARCHAR(50)
);
INSERT INTO entities (entity, code)
SELECT DISTINCT Entity, Code
FROM infectious_cases;

CREATE TABLE infectious_cases_normalized (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity_id INT NOT NULL,
    year YEAR,
    Number_yaws TEXT,
    polio_cases TEXT,
    cases_guinea_worm TEXT,
    Number_rabies TEXT,
    Number_malaria TEXT,
    Number_hiv TEXT,
    Number_tuberculosis TEXT,
    Number_smallpox TEXT,
    Number_cholera_cases TEXT,
    FOREIGN KEY (entity_id) REFERENCES entities(id)
);

INSERT INTO infectious_cases_normalized (
    entity_id,
    year,
    Number_yaws,
    polio_cases,
    cases_guinea_worm,
    Number_rabies,
    Number_malaria,
    Number_hiv,
    Number_tuberculosis,
    Number_smallpox,
    Number_cholera_cases
)
SELECT
    e.id,
    ic.Year,
    ic.Number_yaws,
    ic.polio_cases,
    ic.cases_guinea_worm,
    ic.Number_rabies,
    ic.Number_malaria,
    ic.Number_hiv,
    ic.Number_tuberculosis,
    ic.Number_smallpox,
    ic.Number_cholera_cases
FROM infectious_cases ic
JOIN entities e
    ON ic.Entity = e.entity
   AND (ic.Code = e.code OR (ic.Code IS NULL AND e.code IS NULL));

SELECT COUNT(*)
FROM infectious_cases;

SELECT
    e.entity,
    e.code,
    AVG(CAST(icn.Number_rabies AS DECIMAL(20,4))) AS avg_rabies,
    MIN(CAST(icn.Number_rabies AS DECIMAL(20,4))) AS min_rabies,
    MAX(CAST(icn.Number_rabies AS DECIMAL(20,4))) AS max_rabies,
    SUM(CAST(icn.Number_rabies AS DECIMAL(20,4))) AS sum_rabies
FROM infectious_cases_normalized icn
JOIN entities e ON icn.entity_id = e.id
WHERE icn.Number_rabies <> ''
GROUP BY e.entity, e.code
ORDER BY avg_rabies DESC
LIMIT 10;

SELECT
    id,
    year,
    STR_TO_DATE(CONCAT(year, '-01-01'), '%Y-%m-%d') AS first_day_of_year,
    CURDATE() AS current_date_value,
    TIMESTAMPDIFF(
        YEAR,
        STR_TO_DATE(CONCAT(year, '-01-01'), '%Y-%m-%d'),
        CURDATE()
    ) AS year_diff
FROM infectious_cases_normalized;

DELIMITER //

CREATE FUNCTION year_difference(input_year YEAR)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN TIMESTAMPDIFF(
        YEAR,
        STR_TO_DATE(CONCAT(input_year, '-01-01'), '%Y-%m-%d'),
        CURDATE()
    );
END //

DELIMITER ;

SELECT
    id,
    year,
    year_difference(year) AS year_diff
FROM infectious_cases_normalized;