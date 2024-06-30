# Online Bookstore


## Mid-Term
### Team Members
1. Bibek Rai
2. Rishi Chourasia
---------------
### Tables required and type of attributes
1. Customer Table: For storing data of customers.
* Attributes: CustomerID (Integer), FirstName (String), LastName(String), Email (String), Password(String)
2. Author Table: For storing data for authors.
* Attributes: AuthorID (Integer), Name (String), Email(String), Phone (Integer) 
3. Books Table:For storing data of books.
* Attributes: BookID (Integer), Title (String), AuthorID (Integer), Genre (String), Price (Float), PublicationDate(Date), Rating(Float)
4. Review Table: For storing feedbck from the customers.
* Attributes: ReviewID (Integer), BookID (Integer), CustomerID(Integer), UserComment(String), RatingNum(Integer), ReviewTime(Date)
5. Sales Table: For storing informtion of books sale.
* Attributes: SalesID (Integer), CustomerID (Integer), BookID (Integer) SaleTime(Date), SaleAmount(Float), Quantity (Integer)
6. BookGenre: keeps track of how many books a writer has written in a particular genre during a given time frame.
* Attributes: Genre (String), AuthorID (Integer), BookNum(Integer)

### Creating Tables
``` 
CREATE TABLE Customer (
    CustomerID INTEGER PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(320) UNIQUE NOT NULL,
    Password VARCHAR(255) NOT NULL
);
 
CREATE TABLE Author (
    AuthorID INTEGER PRIMARY KEY,
    Name VARCHAR(100) NOT NULL,
    Email VARCHAR(320),
    Phone VARCHAR(20)
);

CREATE TABLE Books (
    BookID INTEGER PRIMARY KEY,
    Title VARCHAR(255) NOT NULL,
    AuthorID INTEGER,
    Genre VARCHAR(50),
    Price FLOAT,
    PublicationDate DATE,
    Rating FLOAT,
    FOREIGN KEY (AuthorID) REFERENCES Author(AuthorID)
);

CREATE TABLE Review (
    ReviewID INTEGER PRIMARY KEY,
    BookID INTEGER,
    CustomerID INTEGER,
    UserComment TEXT,
    RatingNum INTEGER,
    ReviewTime DATE,
    FOREIGN KEY (BookID) REFERENCES Books(BookID),
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID)
);

CREATE TABLE Sales (
    SalesID INTEGER PRIMARY KEY,
    CustomerID INTEGER,
    BookID INTEGER,
    SaleTime DATE,
    SaleAmount FLOAT,
    Quantity INTEGER,
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID),
    FOREIGN KEY (BookID) REFERENCES Books(BookID)
);

CREATE TABLE BookGenre (
    Genre VARCHAR(50),
    AuthorID INTEGER,
    BookNum INTEGER,
    PRIMARY KEY (Genre, AuthorID),
    FOREIGN KEY (AuthorID) REFERENCES Author(AuthorID)
);

```

### CRUDE opertion on Customer Table:

#### Inserting data
```
INSERT INTO Customer (CustomerID, FirstName, LastName, Email, Password)
VALUES (1, 'Bibek', 'Rai', 'bibekrai123@gmail.com', 'password465');

```

#### Reading data
```
SELECT * FROM Customer WHERE Firstname ='Bibek' AND Lastname ='Rai';
```
 
#### Update data
```
UPDATE Customer SET FirstName = 'Rishi' WHERE CustomerID =1;
```

#### Delete data
```
DELETE FROM Customer WHERE CustomerID = 1;

```

### Requirements for bookstore
* Power writers (authors) with more than X books in the same genre published within the last X years 
```
SELECT a.Name, b.Genre
FROM Books b
JOIN Author a ON b.AuthorID = a.AuthorID
WHERE b.PublicationDate >= DATE_SUB(CURRENT_DATE, INTERVAL 5 YEAR)
GROUP BY a.Name, b.Genre
HAVING COUNT(b.BookID) > 3;
```

* Loyal Customers who has spent more than X dollars in the last year
```
SELECT c.FirstName, c.LastName, c.Email
FROM Customer c
WHERE c.CustomerID IN (
  SELECT s.CustomerID
  FROM Sales s
  WHERE s.SaleTime >= DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR)
  GROUP BY s.CustomerID
  HAVING SUM(s.SaleAmount) > 50
);

```
* Well Reviewed books that has a better user rating than average
```
SELECT b.Title, b.Rating
FROM Books b
WHERE b.Rating > (
    SELECT AVG(b2.Rating)
    FROM Books b2
    INNER JOIN Review r ON b2.BookID = r.BookID
)
ORDER BY b.Rating DESC;

```
* The most popular genre by sales
```
SELECT bg.Genre, COUNT(*) AS TotalSalesCount
FROM Books b
JOIN BookGenre bg ON b.AuthorID = bg.AuthorID
JOIN Sales s ON b.BookID = s.BookID
GROUP BY bg.Genre
ORDER BY TotalSalesCount DESC
LIMIT 1;

```

* The 10 most recent posted reviews by Customers 
```
SELECT r.ReviewID, c.FirstName, c.LastName, r.UserComment, r.RatingNum, r.ReviewTime
FROM Review r
JOIN Customer c ON r.CustomerID = c.CustomerID
ORDER BY r.ReviewTime DESC
LIMIT 10;
```

### Typescript interface

##### For Customer interface
```
interface Customer{
  CustomerID: number;
  FirstName: string;
  LastName: string;
  Email: string;
  Password: string;
}
```
##### For Author interface
```
interface Author{
  AuthorID: number;
  Name: string;
  Email: string;
  Phone: string;
}
```
##### For Books interface
```
interface Author{
  BookID: number;
  Title: string;
  AuthorID: number;
  Genre: string;
  Price: number;
  PublicationDate: string;
  Rating: number;
}
```
##### For Review interface

```
interface Review{
  ReviewID: number;
  BookID: number;
  CustomerID: number;
  UserComment: string;
  RatingNum: number;
  ReviewTime: string;
}
```
##### For Sales interface

```
interface Sales{
  SalesID: number;
  CustomerID: number;
  BookID: number;
  SaleTime: string;
  SaleAmount: number;
  Quantity: number;
}
```
##### For BookGenre interface

```
interface BookGenre{
 Genre: string;
  AuthorID: number;
  BookNum: number;
```

#### For modification:

```

--For Customer:

async function createCustomerTbl(firstName: string, lastName: string, email: string, password: string) {
  return db.one(`INSERT INTO Customer (FirstName, LastName, Email, Password) VALUES ($1, $2, $3, $4) RETURNING *`, [firstName, lastName, email, password]);
}

async function readCustomerTbl(customerID: number) {
  return db.one(`SELECT * FROM Customer WHERE CustomerID = $1`, [customerID]);
}

async function updateCustomerTbl(customerID: number, firstName: string, lastName: string, email: string, password: string) {
  return db.one(`UPDATE Customer SET 
                  FirstName = $2, 
                  LastName = $3, 
                  Email = $4, 
                  Password = $5 
                  WHERE CustomerID = $1 RETURNING *`, [customerID, firstName, lastName, email, password]);
}

async function deleteCustomerTbl(customerID: number) {
  return db.one(`DELETE FROM Customer WHERE CustomerID = $1 RETURNING *`, [customerID]);
}
```


--For Author:
```
async function createAuthorTbl(name: string, email: string, phone: string) {
  return db.one(`INSERT INTO Author (Name, Email, Phone) VALUES ($1, $2, $3) RETURNING *`, [name, email, phone]);
}

async function readAuthorTbl(authorID: number) {
  return db.one(`SELECT * FROM Author WHERE AuthorID = $1`, [authorID]);
}

async function updateAuthorTbl(authorID: number, name: string, email: string, phone: string) {
  return db.one(`UPDATE Author SET 
                  Name = $2, 
                  Email = $3, 
                  Phone = $4 
                  WHERE AuthorID = $1 RETURNING *`, [authorID, name, email, phone]);
}

async function deleteAuthorTbl(authorID: number) {
  return db.one(`DELETE FROM Author WHERE AuthorID = $1 RETURNING *`, [authorID]);
}
```


--For Books:
```
async function createBookTbl(title: string, authorID: number, genre: string, price: number, publicationDate: Date, rating: number) {
  return db.one(`INSERT INTO Books (Title, AuthorID, Genre, Price, PublicationDate, Rating) 
                  VALUES ($1, $2, $3, $4, $5, $6) RETURNING *`, 
                  [title, authorID, genre, price, publicationDate, rating]);
}

async function readBookTbl(bookID: number) {
  return db.one(`SELECT * FROM Books WHERE BookID = $1`, [bookID]);
}

async function updateBookTbl(bookID: number, title: string, authorID: number, genre: string, price: number, publicationDate: Date, rating: number) {
  return db.one(`UPDATE Books SET 
                  Title = $2, 
                  AuthorID = $3, 
                  Genre = $4, 
                  Price = $5, 
                  PublicationDate = $6, 
                  Rating = $7 
                  WHERE BookID = $1 RETURNING *`, 
                  [bookID, title, authorID, genre, price, publicationDate, rating]);
}

async function deleteBookTbl(bookID: number) {
  return db.one(`DELETE FROM Books WHERE BookID = $1 RETURNING *`, [bookID]);
}
```


--For Review:
```
async function createReviewTbl(bookID: number, customerID: number, userComment: string, ratingNum: number, reviewTime: Date) {
  return db.one(`INSERT INTO Review (BookID, CustomerID, UserComment, RatingNum, ReviewTime) 
                  VALUES ($1, $2, $3, $4, $5) RETURNING *`, 
                  [bookID, customerID, userComment, ratingNum, reviewTime]);
}

async function readReviewTbl(reviewID: number) {
  return db.one(`SELECT * FROM Review WHERE ReviewID = $1`, [reviewID]);
}

async function updateReviewTbl(reviewID: number, bookID: number, customerID: number, userComment: string, ratingNum: number, reviewTime: Date) {
  return db.one(`UPDATE Review SET 
                  BookID = $2, 
                  CustomerID = $3, 
                  UserComment = $4, 
                  RatingNum = $5, 
                  ReviewTime = $6 
                  WHERE ReviewID = $1 RETURNING *`, 
                  [reviewID, bookID, customerID, userComment, ratingNum, reviewTime]);
}

async function deleteReviewTbl(reviewID: number) {
  return db.one(`DELETE FROM Review WHERE ReviewID = $1 RETURNING *`, [reviewID]);
}
```


-- For Sales
```
async function createSaleTbl(customerID: number, bookID: number, saleTime: Date, saleAmount: number, quantity: number) {
  return db.one(`INSERT INTO Sales (CustomerID, BookID, SaleTime, SaleAmount, Quantity) 
                  VALUES ($1, $2, $3, $4, $5) RETURNING *`, 
                  [customerID, bookID, saleTime, saleAmount, quantity]);
}

async function readSaleTbl(salesID: number) {
  return db.one(`SELECT * FROM Sales WHERE SalesID = $1`, [salesID]);
}

async function updateSaleTbl(salesID: number, customerID: number, bookID: number, saleTime: Date, saleAmount: number, quantity: number) {
  return db.one(`UPDATE Sales SET 
                  CustomerID = $2, 
                  BookID = $3, 
                  SaleTime = $4, 
                  SaleAmount = $5, 
                  Quantity = $6 
                  WHERE SalesID = $1 RETURNING *`, 
                  [salesID, customerID, bookID, saleTime, saleAmount, quantity]);
}

async function deleteSaleTbl(salesID: number) {
  return db.one(`DELETE FROM Sales WHERE SalesID = $1 RETURNING *`, [salesID]);
}

```

-- For BookGenre
```
async function createBookGenreTbl(genre: string, authorID: number, bookNum: number) {
  return db.one(`INSERT INTO BookGenre (Genre, AuthorID, BookNum) 
                  VALUES ($1, $2, $3) RETURNING *`, 
                  [genre, authorID, bookNum]);
}

async function readBookGenreTbl(genre: string, authorID: number) {
  return db.one(`SELECT * FROM BookGenre WHERE Genre = $1 AND AuthorID = $2`, [genre, authorID]);
}

async function updateBookGenreTbl(genre: string, authorID: number, bookNum: number) {
  return db.one(`UPDATE BookGenre SET 
                  BookNum = $3 
                  WHERE Genre = $1 AND AuthorID = $2 RETURNING *`, 
                  [genre, authorID, bookNum]);
}

async function deleteBookGenreTbl(genre: string, authorID: number) {
  return db.one(`DELETE FROM BookGenre WHERE Genre = $1 AND AuthorID = $2 RETURNING *`, [genre, authorID]);
}
```






