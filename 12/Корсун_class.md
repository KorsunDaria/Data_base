## 1. Больница

```sql
CREATE SCHEMA hospital;

CREATE TABLE hospital.department (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE hospital.room (
    id            INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    department_id INTEGER NOT NULL REFERENCES hospital.department(id),
    number        VARCHAR(10) NOT NULL,
    UNIQUE (department_id, number)
);

CREATE TABLE hospital.doctor (
    id               INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    department_id    INTEGER NOT NULL REFERENCES hospital.department(id),
    full_name        VARCHAR(150) NOT NULL,
    address          VARCHAR(250) NOT NULL,
    birth_date       DATE NOT NULL,
    passport         VARCHAR(20) NOT NULL UNIQUE,
    specialty        VARCHAR(100) NOT NULL,
    experience_years INTEGER NOT NULL CHECK (experience_years >= 0)
);

CREATE TABLE hospital.patient (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name  VARCHAR(150) NOT NULL,
    birth_date DATE NOT NULL,
    policy     VARCHAR(20) NOT NULL UNIQUE
);

CREATE TABLE hospital.hospitalization (
    id             INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    patient_id     INTEGER NOT NULL REFERENCES hospital.patient(id),
    doctor_id      INTEGER NOT NULL REFERENCES hospital.doctor(id),
    room_id        INTEGER NOT NULL REFERENCES hospital.room(id),
    admission_date DATE NOT NULL,
    discharge_date DATE,
    diagnosis      VARCHAR(250) NOT NULL,
    CHECK (discharge_date IS NULL OR discharge_date >= admission_date)
);
```

## 2. Повышение квалификации

```sql
CREATE SCHEMA training;

CREATE TABLE training.employee (
    id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name VARCHAR(150) NOT NULL
);

CREATE TABLE training.course (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       VARCHAR(200) NOT NULL,
    start_date DATE NOT NULL,
    end_date   DATE NOT NULL,
    teacher_id INTEGER NOT NULL REFERENCES training.employee(id),
    CHECK (end_date >= start_date)
);

CREATE TABLE training.enrollment (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    employee_id INTEGER NOT NULL REFERENCES training.employee(id),
    course_id   INTEGER NOT NULL REFERENCES training.course(id),
    UNIQUE (employee_id, course_id)
);

CREATE TABLE training.certificate (
    id            INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    enrollment_id INTEGER NOT NULL UNIQUE REFERENCES training.enrollment(id),
    number        VARCHAR(30) NOT NULL UNIQUE,
    issue_date    DATE NOT NULL
);
```

## 3. Библиотека

```sql
CREATE SCHEMA library;

CREATE TABLE library.branch (
    id     INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name   VARCHAR(100) NOT NULL,
    street VARCHAR(100) NOT NULL,
    house  VARCHAR(20) NOT NULL
);

CREATE TABLE library.publisher (
    id      INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name    VARCHAR(150) NOT NULL,
    address VARCHAR(250) NOT NULL
);

CREATE TABLE library.genre (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE library.book (
    isbn         VARCHAR(17) PRIMARY KEY,
    title        VARCHAR(250) NOT NULL,
    publisher_id INTEGER NOT NULL REFERENCES library.publisher(id),
    cover_type   VARCHAR(20) NOT NULL
                 CHECK (cover_type IN ('мягкая', 'твердая', 'суперобложка')),
    year         INTEGER NOT NULL,
    pages        INTEGER NOT NULL CHECK (pages > 0)
);

CREATE TABLE library.book_genre (
    book_isbn VARCHAR(17) NOT NULL REFERENCES library.book(isbn),
    genre_id  INTEGER NOT NULL REFERENCES library.genre(id),
    PRIMARY KEY (book_isbn, genre_id)
);

CREATE TABLE library.copy (
    id               INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    book_isbn        VARCHAR(17) NOT NULL REFERENCES library.book(isbn),
    branch_id        INTEGER NOT NULL REFERENCES library.branch(id),
    inventory_number VARCHAR(30) NOT NULL UNIQUE
);

CREATE TABLE library.reader (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name  VARCHAR(150) NOT NULL,
    birth_year INTEGER NOT NULL,
    address    VARCHAR(250) NOT NULL
);

CREATE TABLE library.loan (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    copy_id     INTEGER NOT NULL REFERENCES library.copy(id),
    reader_id   INTEGER NOT NULL REFERENCES library.reader(id),
    issue_date  DATE NOT NULL,
    due_date    DATE NOT NULL,
    return_date DATE,
    CHECK (due_date >= issue_date),
    CHECK (return_date IS NULL OR return_date >= issue_date)
);

CREATE UNIQUE INDEX ux_loan_copy_on_hands
ON library.loan (copy_id)
WHERE return_date IS NULL;
```

## 4. Аптека

```sql
CREATE SCHEMA pharmacy;

CREATE TABLE pharmacy.manufacturer (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(150) NOT NULL
);

CREATE TABLE pharmacy.drug_type (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE pharmacy.drug (
    id              INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name            VARCHAR(200) NOT NULL,
    manufacturer_id INTEGER NOT NULL REFERENCES pharmacy.manufacturer(id),
    type_id         INTEGER NOT NULL REFERENCES pharmacy.drug_type(id),
    otc             BOOLEAN NOT NULL
);

CREATE TABLE pharmacy.price (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    drug_id    INTEGER NOT NULL REFERENCES pharmacy.drug(id),
    value      NUMERIC(10, 2) NOT NULL CHECK (value >= 0),
    date_from  DATE NOT NULL,
    date_to    DATE,
    CHECK (date_to IS NULL OR date_to >= date_from)
);

CREATE TABLE pharmacy.supplier (
    id      INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name    VARCHAR(150) NOT NULL,
    inn     VARCHAR(12) NOT NULL UNIQUE,
    address VARCHAR(250) NOT NULL
);

CREATE TABLE pharmacy.branch (
    id      INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name    VARCHAR(100) NOT NULL,
    address VARCHAR(250) NOT NULL
);

CREATE TABLE pharmacy.delivery (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    supplier_id INTEGER NOT NULL REFERENCES pharmacy.supplier(id),
    branch_id   INTEGER NOT NULL REFERENCES pharmacy.branch(id),
    date        DATE NOT NULL
);

CREATE TABLE pharmacy.delivery_item (
    id             INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    delivery_id    INTEGER NOT NULL REFERENCES pharmacy.delivery(id),
    drug_id        INTEGER NOT NULL REFERENCES pharmacy.drug(id),
    quantity       INTEGER NOT NULL CHECK (quantity > 0),
    purchase_price NUMERIC(10, 2) NOT NULL CHECK (purchase_price >= 0)
);

CREATE TABLE pharmacy.customer (
    id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name VARCHAR(150) NOT NULL,
    phone     VARCHAR(20) NOT NULL
);

CREATE TABLE pharmacy.sale (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    branch_id   INTEGER NOT NULL REFERENCES pharmacy.branch(id),
    customer_id INTEGER REFERENCES pharmacy.customer(id),
    date        DATE NOT NULL
);

CREATE TABLE pharmacy.sale_item (
    id             INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sale_id        INTEGER NOT NULL REFERENCES pharmacy.sale(id),
    drug_id        INTEGER NOT NULL REFERENCES pharmacy.drug(id),
    quantity       INTEGER NOT NULL CHECK (quantity > 0),
    price          NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    prescription_number VARCHAR(30)
);
```

## 5. Предприятие

```sql
CREATE SCHEMA company;

CREATE TABLE company.employee (
    id               INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name        VARCHAR(150) NOT NULL,
    birth_date       DATE NOT NULL,
    experience_years INTEGER NOT NULL CHECK (experience_years >= 0)
);

CREATE TABLE company.education (
    id              INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    employee_id     INTEGER NOT NULL REFERENCES company.employee(id),
    level           VARCHAR(50) NOT NULL,
    institution     VARCHAR(200) NOT NULL,
    graduation_year INTEGER NOT NULL,
    specialty       VARCHAR(150) NOT NULL
);

CREATE TABLE company.department (
    number INTEGER PRIMARY KEY,
    name   VARCHAR(100) NOT NULL
);

CREATE TABLE company.position (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE company.assignment (
    id              INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    employee_id     INTEGER NOT NULL REFERENCES company.employee(id),
    position_id     INTEGER NOT NULL REFERENCES company.position(id),
    department_number INTEGER NOT NULL REFERENCES company.department(number),
    salary          NUMERIC(12, 2) NOT NULL CHECK (salary >= 0),
    order_number    VARCHAR(30) NOT NULL,
    order_date      DATE NOT NULL,
    start_date      DATE NOT NULL
);

CREATE TABLE company.vacation (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    employee_id INTEGER NOT NULL REFERENCES company.employee(id),
    start_date  DATE NOT NULL,
    end_date    DATE NOT NULL,
    CHECK (end_date >= start_date)
);

CREATE TABLE company.sick_leave (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    employee_id INTEGER NOT NULL REFERENCES company.employee(id),
    number      VARCHAR(30) NOT NULL UNIQUE
);

CREATE TABLE company.disability_period (
    id            INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sick_leave_id INTEGER NOT NULL REFERENCES company.sick_leave(id),
    start_date    DATE NOT NULL,
    end_date      DATE NOT NULL,
    CHECK (end_date >= start_date)
);
```

## 6. Пиццерия

```sql
CREATE SCHEMA pizzeria;

CREATE TABLE pizzeria.branch (
    id      INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name    VARCHAR(100) NOT NULL,
    address VARCHAR(250) NOT NULL
);

CREATE TABLE pizzeria.client (
    id    INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name  VARCHAR(100) NOT NULL,
    phone VARCHAR(20) NOT NULL UNIQUE
);

CREATE TABLE pizzeria.courier (
    id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    branch_id INTEGER NOT NULL REFERENCES pizzeria.branch(id),
    full_name VARCHAR(150) NOT NULL,
    phone     VARCHAR(20) NOT NULL
);

CREATE TABLE pizzeria.pizza (
    id    INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name  VARCHAR(100) NOT NULL UNIQUE,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0)
);

CREATE TABLE pizzeria.product (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    unit VARCHAR(20) NOT NULL
);

CREATE TABLE pizzeria.pizza_composition (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    pizza_id   INTEGER NOT NULL REFERENCES pizzeria.pizza(id),
    product_id INTEGER NOT NULL REFERENCES pizzeria.product(id),
    quantity   NUMERIC(10, 3) NOT NULL CHECK (quantity > 0),
    UNIQUE (pizza_id, product_id)
);

CREATE TABLE pizzeria."order" (
    id               INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    client_id        INTEGER NOT NULL REFERENCES pizzeria.client(id),
    branch_id        INTEGER NOT NULL REFERENCES pizzeria.branch(id),
    courier_id       INTEGER REFERENCES pizzeria.courier(id),
    created_at       TIMESTAMP NOT NULL,
    order_method     VARCHAR(10) NOT NULL CHECK (order_method IN ('phone', 'internet')),
    delivery_address VARCHAR(250) NOT NULL,
    status           VARCHAR(30) NOT NULL
);

CREATE TABLE pizzeria.order_item (
    id       INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES pizzeria."order"(id),
    pizza_id INTEGER NOT NULL REFERENCES pizzeria.pizza(id),
    quantity INTEGER NOT NULL CHECK (quantity > 0)
);
```
