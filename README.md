# fcc-periodic-table

This project involves managing a PostgreSQL database that contains information about chemical elements. The project is divided into three parts: fixing the database, creating a Git repository, and writing a script to query the database.

## Table of Contents

- [Introduction](#introduction)
- [Project Structure](#project-structure)
- [Setup](#setup)
- [Usage](#usage)
- [Database Fixes](#database-fixes)
- [Script Details](#script-details)
- [Contributing](#contributing)
- [License](#license)

## Introduction

You are provided with a `periodic_table` database that has information about some chemical elements. Your tasks are to fix errors in the database, create a Git repository, and write a script that accepts an argument (atomic number, symbol, or name of an element) and outputs information about that element.

## Project Structure

periodic_table/
├── element.sh
├── periodic_table.sql
└── README.md


- `element.sh`: The script to query the database.
- `periodic_table.sql`: SQL dump file to rebuild the database.
- `README.md`: This file.

## Setup

### Prerequisites

- PostgreSQL
- Git

### Database Setup

1. **Connect to the Database**:
   ```bash
   psql --username=freecodecamp --dbname=periodic_table
   ```


2. **Load the Database**: If you need to rebuild the database, use the following command:
```bash
psql -U postgres < periodic_table.sql
```

## Usage
Running the Script

The script element.sh accepts an argument (atomic number, symbol, or name of an element) and outputs information about the element.

```bash
./element.sh <argument>
```
**Examples**:
```bash

./element.sh 1        # Using atomic number
./element.sh H        # Using symbol
./element.sh Hydrogen # Using name
```

**Sample Output**:

The element with atomic number 1 is Hydrogen (H). It's a nonmetal, with a mass of 1.008 amu. Hydrogen has a melting point of -259.16 celsius and a boiling point of -252.87 celsius.


### Database Fixes
The following SQL commands were used to fix and update the database:

```sql
-- Rename Columns
ALTER TABLE properties RENAME COLUMN weight TO atomic_mass;
ALTER TABLE properties RENAME COLUMN melting_point TO melting_point_celsius;
ALTER TABLE properties RENAME COLUMN boiling_point TO boiling_point_celsius;

-- Set NOT NULL Constraints
ALTER TABLE properties ALTER COLUMN melting_point_celsius SET NOT NULL;
ALTER TABLE properties ALTER COLUMN boiling_point_celsius SET NOT NULL;

-- Add UNIQUE and NOT NULL Constraints
ALTER TABLE elements ADD CONSTRAINT unique_symbol UNIQUE(symbol);
ALTER TABLE elements ADD CONSTRAINT unique_name UNIQUE(name);
ALTER TABLE elements ALTER COLUMN symbol SET NOT NULL;
ALTER TABLE elements ALTER COLUMN name SET NOT NULL;

-- Set Foreign Key
ALTER TABLE properties ADD CONSTRAINT fk_atomic_number FOREIGN KEY (atomic_number) REFERENCES elements(atomic_number);

-- Create Types Table
CREATE TABLE types (
  type_id SERIAL PRIMARY KEY,
  type VARCHAR NOT NULL
);

INSERT INTO types (type) VALUES ('metal'), ('nonmetal'), ('metalloid');

-- Add Type ID to Properties Table
ALTER TABLE properties ADD COLUMN type_id INT NOT NULL;
ALTER TABLE properties ADD CONSTRAINT fk_type_id FOREIGN KEY (type_id) REFERENCES types(type_id);

-- Update Properties Table with Type IDs
UPDATE properties SET type_id = (SELECT type_id FROM types WHERE type = properties.type);
ALTER TABLE properties DROP COLUMN type;

-- Capitalize Symbols
UPDATE elements SET symbol = INITCAP(symbol);

-- Remove Trailing Zeros from Atomic Mass
ALTER TABLE properties ALTER COLUMN atomic_mass TYPE DECIMAL;

-- Add Fluorine and Neon
INSERT INTO elements (atomic_number, name, symbol) VALUES (9, 'Fluorine', 'F'), (10, 'Neon', 'Ne');
INSERT INTO properties (atomic_number, atomic_mass, melting_point_celsius, boiling_point_celsius, type_id) VALUES
(9, 18.998, -220, -188.1, (SELECT type_id FROM types WHERE type = 'nonmetal')),
(10, 20.18, -248.6, -246.1, (SELECT type_id FROM types WHERE type = 'nonmetal'));

-- Delete Non-Existent Element
DELETE FROM properties WHERE atomic_number = 1000;
DELETE FROM elements WHERE atomic_number = 1000;
```

**Script Details**:
The element.sh script uses the following logic:

- Check if an argument is provided.
- Determine if the argument is an atomic number, symbol, or name.
- Query the database based on the argument.
- Output the element's details if found, or an error message if not.


## Contributing
Contributions are welcome! Please fork the repository and submit a pull request.



