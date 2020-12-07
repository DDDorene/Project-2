# Project-2
Names and UNIs: Yuhan Liu yl4347 , Donglin Guan 3128
Monday, December 7,2020

# The database can be accessed through UNI of dg3128.

# 1 Adding array attributes
The first modification is adding arrays. In the new table credit_due, due_balances store the information that we are due in 1 year, 2years and 3 years respectively. 

```sh
CREATE TABLE credit_due(
       pid  INTEGER,
       credit_id  VARCHAR(10),
       due_balances  REAL[],#
       primary key (pid, credit_id),
       FOREIGN KEY (pid)
               	REFERENCES Passengers);
```

This query searches all the due_balances which satisfy the requirement that the due amount for year is less than 10000.

```sh
SELECT due_balances FROM credit_due where due_balances[1]< 10000
```

# 2 Adding text attributes
The second modification is adding text attribute to our existing datasets of has_searches and rates reviews. We change some of the existing attributes in these tables to text type.

```sh
ALTER TABLE rates_Reviews
    ALTER COLUMN remarks TYPE TEXT,
    ALTER COLUMN source TYPE TEXT;
```
```sh   
ALTER TABLE has_searches
    ALTER COLUMN search_keyword TYPE TEXT;
```
This query searches the word ‘good’ in the search_keyword from the table has_searches.

```sh
SELECT search_keyword FROM has_searches where to_tsvector( search_keyword) @@ to_tsquery (‘good’)
```

# 3 Creating trigger and trigger function
The third modification is adding a trigger to the table rate_Reviews. This trigger ensure that any insert, update or delete of a row in the rates_Reviews table is recorded in Reviews_audit table. The updated time is recorded together with the type of operation performed on it. And it checks that reviewnumber, Flightnumber, rating, remarks and source is given and that the salary is a positive value.

```sh
CREATE TABLE Reviews_audit( 
    Operation   char(1)   NOT NULL, 
    last_updated   timestamp,
    reviewsnumber INTEGER  NOT NULL,
    rating  INTEGER NOT NULL,
    remarks   TEXT,
    source    TEXT,
    flightnumber VARCHAR(10),
    PRIMARY KEY (reviewsnumber, flightnumber),
    FOREIGN KEY (flightnumber)
                	REFERENCES Flights

CREATE FUNCTION process_Reviews_audit() RETURNS TRIGGER AS $Reviews_audit$
    BEGIN
        IF NEW.reviewnumber IS NULL THEN
            RAISE EXCEPTION 'rating cannot be null';END IF;
        IF NEW.rating IS NULL THEN
            RAISE EXCEPTION 'rating cannot be null';END IF;
        IF NEW.remarks IS NULL THEN
            RAISE EXCEPTION 'remarks cannot be null';END IF;
        IF NEW.source IS NULL THEN
            RAISE EXCEPTION 'source cannot be null';END IF;
        IF NEW.flightnumber IS NULL THEN
            RAISE EXCEPTION 'flightnumber cannot be null';END IF;

        IF (TG_OP = 'DELETE') THEN
            INSERT INTO Reviews_audit SELECT 'D', now(),OLD.*;
        ELSIF (TG_OP = 'UPDATE') THEN
            INSERT INTO Reviews_audit SELECT 'U', now(),NEW.*;
        ELSIF (TG_OP = 'INSERT') THEN
            INSERT INTO Reviews_audit SELECT 'I', now(),NEW.*;
        END IF;
        RETURN NEW;
    END;
$Reviews_audit$ LANGUAGE plpgsql;

CREATE TRIGGER Reviews_audit
AFTER INSERT OR UPDATE OR DELETE ON rates_Reviews FOR EACH ROW EXECUTE FUNCTION process_Reviews_audit();
```

This query delete the passenger with pid = 2 table in rate_Reviews probably because the user with pid = 2 does not want this rating and remarks valid anymore. The row will be removed from the rate_Reviews but will saved in the Reviews_audit recorded with deletion operation,'D', and the time this the being deleted.

```sh
DELETE FROM rate_Reviews WHERE pid = 2
```

