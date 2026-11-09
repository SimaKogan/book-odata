# Book Library: OData service

A Spring Boot REST service that exposes a book library through the OData v4 protocol (Apache Olingo). Books have credentials (genre, page count) and a reservation status.

## Tech stack

Java 17, Spring Boot 3.5, Spring Data JPA, PostgreSQL, Apache Olingo 5 (OData v4), Lombok, Maven. Tests run on an in-memory H2 database.

## Getting started

You need Java 17 and a running PostgreSQL instance.

```bash
export DB_URL=jdbc:postgresql://localhost:5432/postgres   # optional, this is the default
export DB_USERNAME=postgres                               # optional, this is the default
export DB_PASSWORD=your_password

./mvnw spring-boot:run
```

Tables are created automatically on startup (`spring.jpa.hibernate.ddl-auto=update`), but they start empty. Add a few rows to try the API:

```sql
INSERT INTO book_library (author_surname, author_name, bookname)
VALUES ('Harari', 'Yuval Noah', 'Sapiens');

INSERT INTO book_credential (id_book, book_genre, pages_amount)
SELECT id_book, 'NonFiction', 443 FROM book_library WHERE bookname = 'Sapiens';

INSERT INTO book_status (id_book, reserved_status)
SELECT id_book, false FROM book_library WHERE bookname = 'Sapiens';
```

## Tests

```bash
./mvnw clean test
```

Tests use the `test` profile with an in-memory H2 database, so PostgreSQL is not required.

## API examples

| Request | Description |
| --- | --- |
| `GET /odata/$metadata` | Service metadata |
| `GET /odata/Books` | All books |
| `GET /odata/Books(1)` | Book by id |
| `GET /odata/Books?$select=title` | Only titles |
| `GET /odata/Books?$expand=credential,status` | Books with credentials and status |
| `GET /odata/Books?$expand=credential&$filter=credential/bookGenre eq 'NonFiction'` | Filter by genre |
| `GET /odata/Books?$orderby=title asc` | Sort by title (leading "a", "an", "the" are ignored) |
| `GET /odata/Books(1)/credential` | Credentials of a book |
| `GET /odata/Books(1)/status` | Status of a book |
| `GET /odata/Books?$expand=status&$filter=status/reservedStatus eq true` | Reserved books |

Reserve a book:

```
POST /odata/ReserveBook
Content-Type: application/json

{ "title": "Sapiens" }
```

## Notes

CORS is open for all origins on `/odata/**`. This is fine for a demo, but should be restricted in production.
