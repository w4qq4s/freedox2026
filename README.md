# freedox2026

## a solution that works as a platform that consists of the data of all the equipments and stuff in a institution.

### Core features to make it look different: 
1. QR-driven physical to digital binding mechanism 
    - generate a qr code for every equipment record. a lightweight webapp that can be scanned from phone, a simple lightweight html css js qr reader that leads the user to a kiosk-alike page which mentions status of it.

2. predictive maintenance / "at risk" scoring
    - instead of just functional/non functional flag, add a simple derived risk score based on equipment age, category or product/typical lifespam, and maintenance frequency. "equipment to likely fail soon" type stuff. Not through ML but by a simple transparent weighted formula.

3. smart duplicate suggestions
    - when adding a new equipment, match against the records that are already there and flag possible duplicates.

4. auto generated audit report
    - looking at the use case, ofcourse it's being used for certifications of the institution, so it would be convenient if they can export the reports for every single one of the labs, along with equipments.

### Basic Features:
1. RBAC - 3 roles:
    1. Lab Assistant: 
        - can do basic stuff such as updating lab equipments etc.
    2. Department Head: 
        - can view/edit report of their dept
    3. superadmin:
        - full CRUD across all departments

2. SQL injection prevention
