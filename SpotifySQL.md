NEED TO ADD AVERAGE ENGAGEMENT AS WELL

How many employees currently work?

How many employees have been terminated for any reason?

How many employees does each manager manage?

```sql
	select ManagerName, count(*) from employee where EmploymentStatus = 'Active' group by ManagerName
```

Which managers seem to be doing the best in terms of average employee engagement?



what is the average salary and happiness per department?

select department, count(*), avg(EmpSatisfaction), avg(Salary) from employee where EmploymentStatus = 'Active' group by department

What is the average happiness for people that left and why did they leave?

SELECT
    TermReason,
    COUNT(*) AS EmployeeCount,
    AVG(EmpSatisfaction) AS AvgEmpSatisfaction
FROM
    employee
WHERE
    TermReason != 'N/A-StillEmployed'
GROUP BY
    TermReason
ORDER BY
	EmployeeCount DESC


What are the most common ways for people to apply?

Select
	RecruitmentSource,
    count(*)
from
employee
group by RecruitmentSource
order by count(*) desc

What is the average salary and happiness by sex?

SELECT
    Sex,
    AVG(Salary) AS AvgSalary,
    AVG(EmpSatisfaction) AS AvgEmpSatisfaction
FROM
    employee
WHERE
	EmploymentStatus = 'Active'
GROUP BY
    Sex;



what are the average salary and hapiness by race?

SELECT
    RaceDesc,
    AVG(Salary) AS AvgSalary,
    AVG(EmpSatisfaction) AS AvgEmpSatisfaction
FROM
    employee
GROUP BY
    RaceDesc;


Who deserves a raise?

SELECT
    Employee_Name,
    Salary,
    PerformanceScore,
    Absences,
    EmploymentStatus,
    Department,
    (SELECT AVG(Salary) FROM employee e2 WHERE e2.Department = employee.Department AND e2.EmploymentStatus = 'Active') AS AvgDepartmentSalary
FROM
    employee
WHERE
    Salary < (SELECT AVG(Salary) FROM employee e1 WHERE e1.Department = employee.Department AND e1.EmploymentStatus = 'Active') AND
    PerformanceScore = 'Exceeds' AND
    Absences < (SELECT AVG(Absences) FROM employee WHERE EmploymentStatus = 'Active') AND
    EmploymentStatus = 'Active';


who should be let go?

WITH EmployeeAnalysis AS (
    SELECT
        Employee_Name,
        Salary,
        PerformanceScore,
        Absences,
        EmploymentStatus,
        Department,
        AVG(Salary) OVER (PARTITION BY Department) AS AvgDepartmentSalary
    FROM
        employee
    WHERE
        EmploymentStatus = 'Active'
)

SELECT
    Employee_Name,
    Salary,
    PerformanceScore,
    Absences,
    EmploymentStatus,
    Department,
    AvgDepartmentSalary
FROM
    EmployeeAnalysis
WHERE
    PerformanceScore IN ('Needs Improvement', 'PIP') AND
    Absences > (SELECT AVG(Absences) FROM EmployeeAnalysis) AND
    Salary > AvgDepartmentSalary;


Who should HR pursue to perhaps rehire?

SELECT
    Employee_Name,
    Salary,
    PerformanceScore,
    Absences,
    EmpSatisfaction,
    EmploymentStatus,
    Department
FROM
    employee
WHERE
    EmploymentStatus = 'Voluntarily Terminated' AND
    PerformanceScore = 'Exceeds' AND
    Absences < (SELECT AVG(Absences) FROM employee WHERE EmploymentStatus = 'Voluntarily Terminated' AND PerformanceScore = 'Exceeds') AND
    EmpSatisfaction > (SELECT AVG(EmpSatisfaction) FROM employee WHERE EmploymentStatus = 'Voluntarily Terminated' AND PerformanceScore = 'Exceeds');