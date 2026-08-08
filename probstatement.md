### V11 — Infrastructure & Equipment Inventory
### Business / Accreditation Requirement
Institutions must document laboratories, equipment, facilities and available infrastructure for accreditation review.
### MVP Objective
Build an institutional asset/infrastructure register covering the Department → Laboratory → Equipment hierarchy.
#### Data You Must Capture:
- Department
- Laboratory
- Equipment
- Equipment category
- Quantity
- Purchase year
- Status (functional/non-functional)
- Location
- Maintenance information
#### Required UI Flow
- Infrastructure → Laboratory → Equipment → Equipment Details
- Minimum Expected Output
- Equipment by lab
- Equipment by department
- Functional vs. non-functional equipment
- Category-wise inventory
- Common Core Entities Available
- Institution, Department, Program, Academic Year, Semester, Student, Faculty, Course
ER Design Challenge
Model the hierarchical relationship Department → Laboratory → Equipment correctly, so equipment can be traced back to both its lab and its owning department without duplicating department data at the equipment level.
Dummy Dataset Guidance
At least 3 departments, 6 laboratories, and 40+ equipment records across at least 5 categories, with a mix of functional/non-functional status.
Acceptance Test
A reviewer can drill down from a department to its labs to a specific piece of equipment, and the functional/non-functional and category-wise reports match the underlying records.
Deliverables
Working MVP demonstrating Create → View → Search/Filter → Update → Report/Insight
ER diagram and schema for your vertical extension
docs/technology-decision.md documenting major technology/design decisions
Source code in a GitHub repository with clear commit history
Short live demonstration and defense of engineering decisions
