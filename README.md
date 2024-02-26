# SQL CRUD

# Part 1: Restaurant Finder
1. **Create Table**
```sql
CREATE TABLE restaurants (
    RestaurantName TEXT
    RestaurantID INTEGER PRIMARY KEY,
    Category TEXT NOT NULL,
    PriceTier TEXT NOT NULL,
    Neighborhood TEXT,
    OpenTime TEXT,
    ClosingTime TEXT,
    AverageRating REAL,
    GoodForKids INTEGER
);

CREATE TABLE reviews (
    ReviewID INT AUTO_INCREMENT PRIMARY KEY,
    RestaurantID INT,
    UserID INT,
    Rating INT CHECK (Rating BETWEEN 1 AND 5),
    Comment TEXT,
    ReviewDate DATE,
    FOREIGN KEY (RestaurantID) REFERENCES restaurants(RestaurantID)
);
```
2. **Import csv**
```sql
.mode csv
.import "/Users/yizj/Desktop/Database Design/4-sql-crud-beaverjuly/data/restaurant.csv" restaurants
```
3.**Find all cheap restaurants in a particular neighborhood**

I picked "Downtown" as an example neighborhood:

```sql
SELECT * FROM restaurants
WHERE PriceTier = 'low' AND Neighborhood = 'Upper East Side';
```
4. **Find all restaurants in a particular genre with 3 stars or more, ordered by the number of stars descending**

Choosing "Italian" as the genre example:

```sql
SELECT * FROM restaurants
WHERE Category = 'Italian' AND AverageRating >= 3
ORDER BY AverageRating DESC;

```
3. **Find all restaurants that are open**

```sql
SELECT * FROM restaurants
WHERE strftime('%H:%M', 'now', 'localtime') >= OpeningTime
AND strftime('%H:%M', 'now', 'localtime') <= ClosingTime;

```
4. **Leave a review for a restaurant**

```sql
INSERT INTO reviews (RestaurantID, Rating, Comment, ReviewDate)
VALUES (1, 5, 'Great food and atmosphere!', date('now'));
```

5. **Delete all restaurants that are not good for kids**

```sql
DELETE FROM restaurants
WHERE GoodForKids = 0;
```
6. **Find the number of restaurants in each NYC neighborhood**

```sql
SELECT Neighborhood, COUNT(*) AS NumberOfRestaurants
FROM restaurants
GROUP BY Neighborhood;

```
# Part 2: Social Media Post
1. **Create Table**
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    handle TEXT UNIQUE NOT NULL
);

CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    content TEXT NOT NULL,
    type TEXT NOT NULL CHECK (type IN ('message', 'story')),
    recipient_id INTEGER,
    visibility INTEGER NOT NULL CHECK (visibility IN (0, 1)),
    created_at DATETIME NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (recipient_id) REFERENCES users(id)
);
```

2. **Import csv**

```sql
.mode csv
.import "/Users/yizj/Desktop/Database Design/4-sql-crud-beaverjuly/data/users.csv" users
.import "/Users/yizj/Desktop/Database Design/4-sql-crud-beaverjuly/data/posts.csv" posts
```

3. **Resgister new user**
```sql
INSERT INTO users (email, password, handle) VALUES ('newuser@example.com', 'password123', 'newuser');

```

4. **Create new story**
```sql
INSERT INTO posts (user_id, content, type, recipient_id, visibility, created_at) 
VALUES (1, 'Hello!', 'message', 2, 1, datetime('now'));
```

5. **Create new Story**
```sql
INSERT INTO posts (user_id, content, type, visibility, created_at) 
VALUES (1, 'My first story', 'story', 1, datetime('now'));
```

6. **Show the 10 most recent visible Messages and Stories:**
```sql
SELECT * FROM posts WHERE visibility = 1 ORDER BY created_at DESC LIMIT 10;
```

7. **Show the 10 most recent visible Messages sent by User 1 to User 2:**

```sql
SELECT * FROM posts 
WHERE type = 'message' AND user_id = 1 AND recipient_id = 2 AND visibility = 1 
ORDER BY created_at DESC 
LIMIT 10;
```

8. **Make all Stories older than 24 hours invisible:**
```sql
UPDATE posts SET visibility = 0 
WHERE type = 'story' AND (JULIANDAY('now') - JULIANDAY(created_at)) * 24 > 24;
```

9. **Show all invisible Messages and Stories:**

```sql
SELECT * FROM posts WHERE visibility = 0 ORDER BY created_at DESC;

```

10. **Show the number of posts by each User:**
```sql
SELECT user_id, COUNT(*) as post_count FROM posts GROUP BY user_id;
```

11. **Show the post text and email address of all posts made in the last 24 hours:**
```sql
SELECT p.content, u.email FROM posts p
JOIN users u ON p.user_id = u.id
WHERE (JULIANDAY('now') - JULIANDAY(p.created_at)) * 24 <= 24;
```

12. **Show the email addresses of all Users who have not posted anything yet:**
```sql
SELECT email FROM users WHERE id NOT IN (SELECT DISTINCT user_id FROM posts);
```