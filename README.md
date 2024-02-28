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

I picked "Upper East Side" as an example neighborhood:

```sql
SELECT * FROM restaurants WHERE PriceTier = 'low';
SELECT * FROM restaurants WHERE Neighborhood = 'Upper East Side';
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
WHERE strftime('%H:%M', 'now', 'localtime') >= OpenTime
AND strftime('%H:%M', 'now', 'localtime') <= ClosingTime;
```
4. **Leave a review for a restaurant**

```sql
INSERT INTO reviews (RestaurantID, Rating, Comment, ReviewDate)
VALUES (1, 5, 'Great food and atmosphere!', date('now'));
```
Checking that the review has been successfully inserted

```sql
SELECT * FROM reviews;
```

5. **Delete all restaurants that are not good for kids**

```sql
DELETE FROM restaurants
WHERE GoodForKids = 0;
```
Checking that not-for-kids rows has been successfully deleted
 
```sql
SELECT * FROM restaurants WHERE GoodForKids = 0;
```
And yes indeed nothing is returned!

6. **Find the number of restaurants in each NYC neighborhood**

```sql
SELECT Neighborhood, COUNT(*) AS NumberOfRestaurants
FROM restaurants
GROUP BY Neighborhood;
```
These should be the output:

```sql
Astoria,53
Bushwick,45
Chelsea,38
Greenpoint,49
Harlem,35
"Lower East Side",65
Midtown,45
"Park Slope",46
"Upper East Side",51
Williamsburg,50
```
These altogether add up to 477, which is the number of rows in my restaurants csv after deleting the restaurants that are not good for kids.

Here is the total number of restaurants:

```sql
SELECT COUNT(*) AS TotalNumberOfRestaurants
FROM restaurants;
```
Here is the total number of rows:
```sql
SELECT COUNT(*) AS NumberOfRows FROM restaurants;
```

# Part 2: Social Media Post
1. **Create Table**
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE,
    password TEXT,
    handle TEXT UNIQUE
);

CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER,
    content TEXT,
    type TEXT NOT NULL CHECK (type IN ('message', 'story')),
    recipient_id INTEGER,
    visibility INTEGER,
    created_at DATETIME,
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
Checking that the new user has been successfully inserted

```sql
SELECT * FROM users WHERE handle = 'newuser';
```

4. **Create new story**
```sql
INSERT INTO posts (user_id, content, type, recipient_id, visibility, created_at) 
VALUES (1, 'Hello!', 'message', 2, 1, datetime('now'));
```

Checking whether the post has been successfully inserted:

```sql
SELECT * FROM posts 
WHERE user_id = 1 
AND content = 'Hello!' 
AND type = 'message' 
AND recipient_id = 2 
AND visibility = 1;
```

5. **Create new Story**
```sql
INSERT INTO posts (user_id, content, type, visibility, created_at) 
VALUES (1, 'My first story', 'story', 1, datetime('now'));
```

Again, checking whether the insetion is succesful:
```sql
SELECT * FROM posts 
WHERE user_id = 1 
AND content = 'My first story' 
AND type = 'story' 
AND visibility = 1;
```

6. **Show the 10 most recent visible Messages and Stories:**
```sql
SELECT * FROM posts WHERE visibility = 1 ORDER BY created_at DESC LIMIT 10;
```
These should be the output:
```sql
1002,1,"My first story",story,,1,"2024-02-28 21:13:54"
1001,1,Hello!,message,2,1,"2024-02-28 21:12:28"
53,255,"Ut tellus. Nulla ut erat id mauris vulputate elementum. Nullam varius.",story,402,1,"2024-02-26 22:57:15"
374,551,"Morbi non quam nec dui luctus rutrum. Nulla tellus. In sagittis dui vel nisl.",message,514,1,"2024-02-26 11:22:42"
434,781,"Donec semper sapien a libero. Nam dui. Proin leo odio, porttitor id, consequat in, consequat ut, nulla. Sed accumsan felis. Ut at dolor quis odio consequat varius.",message,981,1,"2024-02-26 02:59:53"
347,709,"Donec posuere metus vitae ipsum. Aliquam non mauris. Morbi non lectus. Aliquam sit amet diam in magna bibendum imperdiet. Nullam orci pede, venenatis non, sodales sed, tincidunt eu, felis. Fusce posuere felis sed lacus.",message,338,1,"2024-02-26 01:35:42"
587,469,"Pellentesque eget nunc. Donec quis orci eget orci vehicula condimentum. Curabitur in libero ut massa volutpat convallis. Morbi odio odio, elementum eu, interdum eu, tincidunt in, leo. Maecenas pulvinar lobortis est. Phasellus sit amet erat. Nulla tempus. Vivamus in felis eu sapien cursus vestibulum.",message,552,1,"2024-02-25 07:26:38"
166,200,"Nam dui. Proin leo odio, porttitor id, consequat in, consequat ut, nulla. Sed accumsan felis.",message,460,1,"2024-02-23 15:05:00"
888,731,"Etiam vel augue. Vestibulum rutrum rutrum neque. Aenean auctor gravida sem.",story,440,1,"2024-02-22 04:44:54"
62,790,"Praesent blandit. Nam nulla. Integer pede justo, lacinia eget, tincidunt eget, tempus vel, pede.",message,448,1,"2024-02-21 18:45:09"
```

7. **Show the 10 most recent visible Messages sent by User 1 to User 2:**

```sql
SELECT * FROM posts 
WHERE type = 'message' AND user_id = 1 AND recipient_id = 2 AND visibility = 1 
ORDER BY created_at DESC 
LIMIT 10;
```
There seems to be only one message that satisfies all of the above conditions in my posts table:

```sql
1001,1,Hello!,message,2,1,"2024-02-28 21:12:28"
```

8. **Make all Stories older than 24 hours invisible:**
```sql
UPDATE posts SET visibility = 0 
WHERE type = 'story' AND (JULIANDAY('now') - JULIANDAY(created_at)) * 24 > 24;
```
Agian, checking if anything is returned:
```sql
SELECT * FROM posts 
WHERE type = 'story' 
AND (JULIANDAY('now') - JULIANDAY(created_at)) * 24 > 24
AND visibility != 0;
```
Indeed, nothing is returned!

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
This is what is returned:
```sql
Hello!,mcolloff0@sciencedaily.com
"My first story",mcolloff0@sciencedaily.com
```

12. **Show the email addresses of all Users who have not posted anything yet:**
```sql
SELECT email FROM users WHERE id NOT IN (SELECT DISTINCT user_id FROM posts);
```