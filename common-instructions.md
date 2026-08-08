1. Ground Rules for Every Team

Every team must:
Use the common institutional core entities where applicable.
Design its own vertical ER extension for its assigned problem.
Build a working UI demonstrating the complete data flow.
Use dummy data (no real institutional data).
Demonstrate Create → View → Search/Filter → Update → Report/Insight wherever relevant.
Choose any technology, framework, database, library, cloud platform or AI-assisted development tool.
Justify every major technology and design decision.
Document all AI/tool usage.
No team will be given the final ER model by the organizers — designing it is part of the challenge.

2. Common Core Entities (Given to All Teams)

Institution, Department, Program, Academic Year, Semester, Student, Faculty, Course
These are conceptual entities only. Each team must design its own vertical extension on top of them for its assigned problem.

3. Technology Selection & Justification

You may use any technology, programming language, framework, database, library, cloud platform, or AI-assisted development tool — but the choice must be a reasoned engineering decision. “AI suggested it” is not a valid justification on its own. Your team is responsible for understanding, testing, validating, integrating and defending every major technology and design decision in your MVP.
Maintain a single file: docs/technology-decision.md
Document only your important decisions, not every small library. For each important decision, cover:
Requirement — what problem were you solving?
Options — what alternatives did you consider?
Evaluation — how did the options compare?
Decision — why did you choose this approach?
Evidence — what supports that the decision was correct?
Example of the expected style of statement:
“Our data has Student → Assessment → Question → CO relationships. We require foreign-key constraints and transaction consistency. We considered SQLite and PostgreSQL. We chose PostgreSQL because…”
You do not need to write a long report. Keep it short: (1) What problem were you solving? (2) What alternatives did you consider? (3) Why did you choose this approach?