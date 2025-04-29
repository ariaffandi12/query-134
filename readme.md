Buatlah sebuah Stored Procedure bernama sp_GetCustomerRentals yang:
Menerima input p_customer_id (INT).
Menampilkan daftar judul film (title) dan tanggal rental (rental_date) dari pelanggan
tersebut.
(Gunakan tabel rental, inventory, dan film.)
QUERY
```
DELIMITER //
CREATE PROCEDURE sp_GetCustomerRentals(
IN p_customer_id INT
)
BEGIN
SELECT
f.title AS 'Judul Film',
r.rental_date AS 'Tanggal Rental'
FROM
rental r
JOIN
inventory i ON r.inventory_id = i.inventory_id
JOIN
film f ON i.film_id = f.film_id
WHERE
r.customer_id = p_customer_id
ORDER BY
r.rental_date DESC;
END //
DELIMITER ;
```
CALL
```
CALL sp_GetCustomerRentals(1);
```
2. Buatlah sebuah Function bernama fn_GetCustomerTotalPayment yang:
Menerima input p_customer_id (INT).
Mengembalikan total seluruh pembayaran (amount) pelanggan tersebut dalam bentuk
DECIMAL.
(Gunakan tabel payment.)
QUERY
```
DELIMITER //
CREATE FUNCTION fn_GetCustomerTotalPayment(
p_customer_id INT
)
RETURNS DECIMAL(10,2)
READS SQL DATA
BEGIN
DECLARE total_amount DECIMAL(10,2);
SELECT
COALESCE(SUM(amount), 0.00) INTO total_amount
FROM
payment
WHERE
customer_id = p_customer_id;
RETURN total_amount;
END //
DELIMITER ;
```
FUNCTION
```
SELECT fn_GetCustomerTotalPayment(1) AS total_payment;
```
3. Buatlah Stored Procedure bernama sp_DeactivateInactiveCustomer yang:
Menerima input p_customer_id (INT).
Jika pelanggan tersebut tidak memiliki transaksi rental dalam 6 bulan terakhir, ubah kolom
active menjadi 0 di tabel customer.
Jika masih aktif bertransaksi, jangan lakukan apa-apa.
(Gunakan tabel rental dan customer.)
QUERY
```
DELIMITER //
CREATE PROCEDURE sp_DeactivateInactiveCustomer(
IN p_customer_id INT
)
BEGIN
DECLARE last_rental_date DATETIME;
-- Mendapatkan tanggal rental terakhir dari pelanggan
SELECT
MAX(rental_date) INTO last_rental_date
FROM
rental
WHERE
customer_id = p_customer_id;
-- Jika tidak ada rental dalam 6 bulan terakhir atau tidak pernah
rental, ubah status active menjadi 0
IF last_rental_date IS NULL OR last_rental_date < (CURDATE() -
INTERVAL 6 MONTH) THEN
 UPDATE customer
 SET active = 0
 WHERE customer_id = p_customer_id;
END IF;
END //
DELIMITER ;
```
CALL
```
-- Memeriksa status aktif customer sebelum menjalankan procedure
SELECT customer_id, first_name, last_name, active FROM customer WHERE
customer_id = 1;
-- Menjalankan procedure untuk mengecek dan menonaktifkan customer
bila tidak aktif
CALL sp_DeactivateInactiveCustomer(1);
-- Memeriksa kembali status customer setelah procedure dijalankan
SELECT customer_id, first_name, last_name, active FROM customer WHERE
customer_id = 1;
```
4. Buat Function fn_GetFilmRentalCount yang:
Menerima p_film_id (INT).
Mengembalikan jumlah total berapa kali film tersebut pernah dirental.
Lalu buat Stored Procedure sp_PromotePopularFilm yang:
Menerima p_film_id (INT).
Jika total rental lebih dari 50 kali, update special_features film tersebut menjadi
'Trailers,Commentaries'.
(Gunakan tabel rental, inventory, film.)
QUERY FUNCTION
```
DELIMITER //
CREATE FUNCTION fn_GetFilmRentalCount(
p_film_id INT
)
RETURNS INT
READS SQL DATA
BEGIN
DECLARE rental_count INT;
SELECT
COUNT(*) INTO rental_count
FROM
rental r
JOIN
inventory i ON r.inventory_id = i.inventory_id
WHERE
i.film_id = p_film_id;
RETURN rental_count;
END //
DELIMITER ;
```
QUERY PROCEDURE
```
DELIMITER //
CREATE PROCEDURE sp_PromotePopularFilm(
IN p_film_id INT
)
BEGIN
DECLARE rental_count INT;
-- Mendapatkan jumlah rental menggunakan function yang sudah
dibuat
SET rental_count = fn_GetFilmRentalCount(p_film_id);
-- Jika jumlah rental lebih dari 50, update special_features
IF rental_count > 50 THEN
UPDATE
film
SET
special_features = 'Trailers,Commentaries'
WHERE
film_id = p_film_id;
END IF;
END //
DELIMITER ;
```
FUNCTION
```
SELECT fn_GetFilmRentalCount(1) AS rental_count;
```
CALL PROCEDURE
```
-- Memeriksa special_features film sebelum menjalankan procedure
SELECT film_id, title, special_features FROM film WHERE film_id = 1;
-- Menjalankan procedure untuk mempromosikan film jika sudah dirental
lebih dari 50 kali
CALL sp_PromotePopularFilm(1);
-- Memeriksa kembali special_features film setelah procedure
dijalankan
SELECT film_id, title, special_features FROM film WHERE film_id = 1;
```
KUIS
A. Query
1. Buatlah query untuk menampilkan data karyawan aktif untuk setiap departemen beserta
informasi besar gaji (salary) terbaru yang diterima karyawan tersebut.
Kolom yang ditampilkan: Department number, Department name, Employee number,
Employee full name (combine first and last name), Salary
Petunjuk: Data karyawan yang masih aktif di suatu departemen ditandai dengan tanggal
to_date yang berisi tahun 9999 pada tabel dept_emp, begitu pula dengan gaji terbaru.
QUERY
```
SELECT d.dept_no AS 'Department number',
 d.dept_name AS 'Department name',
 e.emp_no AS 'Employee number',
 CONCAT(e.first_name, ' ', e.last_name) AS 'Employee full name',
 s.salary AS 'Salary'
FROM departments d
JOIN dept_emp de ON d.dept_no = de.dept_no
JOIN employees e ON de.emp_no = e.emp_no
JOIN salaries s ON e.emp_no = s.emp_no
WHERE de.to_date = '9999-01-01' -- karyawan aktif di departemen
AND s.to_date = '9999-01-01' -- gaji terkini
ORDER BY d.dept_name, e.emp_no;
```
DESKRIPSI
Query ini menggabungkan empat tabel utama (departments, dept_emp, employees,
dan salaries) untuk mendapatkan informasi lengkap tentang karyawan aktif dan gaji
terkininya. Saya menggunakan kondisi to_date = '9999-01-01' pada tabel dept_emp
dan salaries untuk memastikan hanya data karyawan yang masih aktif dan gaji terbaru
yang ditampilkan. Penggunaan CONCAT untuk menggabungkan first_name dan
last_name menghasilkan tampilan nama lengkap karyawan yang lebih rapi, dan hasil
diurutkan berdasarkan departemen dan nomor karyawan untuk memudahkan
pembacaan data.
2. Buatlah query untuk menampilkan data histori jabatan (title) seluruh karyawan beserta
departemen tempat karyawan tersebut bekerja.
Misal: Kazuhide Peha (emp_no: 10018) pernah menjabat sebagai Engineer di departemen
Development dan lanjut sebagai Engineer di departemen Production, kemudian naik
jabatan sebagai Senior Engineer di departemen tersebut (Production).
Kolom yang ditampilkan: Employee number, First Name, Last Name, Title, Title form date,
Title to date, Department name, Department employee from date, Department employee to
date.
Petunjuk: Perhatikan masa berlaku jabatan dan masa berlaku penempatan karyawan di
suatu departemen. Dalam contoh di atas, seorang karyawan dapat berpindah departemen
dengan jabatan yang masih sama dan berubah jabatan di departemen yang sama.
QUERY
```
SELECT t.emp_no AS 'Employee number',
e.first_name AS 'First Name',
e.last_name AS 'Last Name',
t.title AS 'Title',
t.from_date AS 'Title from date',
t.to_date AS 'Title to date',
d.dept_name AS 'Department name',
de.from_date AS 'Department employee from date',
de.to_date AS 'Department employee to date'
FROM titles t
JOIN employees e ON t.emp_no = e.emp_no
JOIN dept_emp de ON e.emp_no = de.emp_no
-- Kondisi untuk memastikan periode jabatan dan periode di
departemen tumpang tindih (overlap)
AND t.from_date <= de.to_date
AND t.to_date >= de.from_date
JOIN departments d ON de.dept_no = d.dept_no
ORDER BY t.emp_no, t.from_date, de.from_date;
```
DESKRIPSI
Query ini menampilkan seluruh riwayat jabatan (titles) untuk setiap karyawan, beserta
informasi departemen tempat mereka bekerja pada periode jabatan tersebut.
Pertama, tabel employees digabungkan dengan titles untuk mendapatkan semua
catatan jabatan per karyawan. Kemudian, hasil ini digabungkan
dengan dept_emp tidak hanya berdasarkan emp_no, tetapi juga dengan
kondisi t.from_date <= de.to_date AND t.to_date >= de.from_date. Kondisi ini penting
untuk memastikan bahwa kita hanya menampilkan departemen (dept_emp) di mana
karyawan tersebut bekerja selama periode waktu mereka memegang jabatan (titles)
tertentu, menangani kasus di mana karyawan bisa pindah departemen atau ganti
jabatan. Terakhir, digabungkan dengan departments untuk mendapatkan nama
departemen. Hasilnya diurutkan agar riwayat per karyawan mudah diikuti.
B. STORED ROUTINES
1. Tabel titles mencatat histori jabatan karyawan. Kolom from_date menunjukkan tanggal mulai
sedangkan kolom to_date menunjukkan tanggal berakhir masa jabatan. Setiap ada perubahan
jabatan pada karyawan maka tabel titles diisi data baru.
Buatlah trigger insert pada tabel titles untuk melakukan insert data ke tabel
dept_manager bila title yang diinsertkan adalah Manager. Pastikan informasi di
departemen mana karyawan tersebut menjadi manager juga sesuai.
QUERY
```
DELIMITER //
CREATE TRIGGER add_dept_manager_after_title_insert
AFTER INSERT ON titles
FOR EACH ROW
BEGIN
DECLARE emp_dept_no CHAR(4);
-- Jika jabatan baru adalah Manager
IF NEW.title = 'Manager' THEN
-- Dapatkan departemen saat ini dari karyawan tersebut
SELECT dept_no INTO emp_dept_no
FROM dept_emp
WHERE emp_no = NEW.emp_no
AND to_date = '9999-01-01'; -- karyawan aktif di departemen
ini
-- Jika ditemukan departemen aktif, tambahkan ke dept_manager
IF emp_dept_no IS NOT NULL THEN
-- Selesaikan masa jabatan manager sebelumnya jika ada
UPDATE dept_manager
SET to_date = NEW.from_date
WHERE dept_no = emp_dept_no
AND to_date = '9999-01-01';

-- Tambahkan manager baru
INSERT INTO dept_manager (emp_no, dept_no, from_date,
to_date)
VALUES (NEW.emp_no, emp_dept_no, NEW.from_date,
NEW.to_date);
END IF;
END IF;
END //
DELIMITER ;
```
DESKRIPSI
Trigger ini secara otomatis menambahkan data ke tabel dept_manager ketika ada
penambahan jabatan 'Manager' di tabel titles. Trigger memeriksa departemen aktif
karyawan tersebut, mengakhiri masa jabatan manager sebelumnya jika ada, dan
menambahkan karyawan tersebut sebagai manager baru dengan periode yang sesuai.
Hal ini menjamin bahwa setiap departemen hanya memiliki satu manager aktif pada
satu waktu dan informasi departemen untuk manager baru selalu konsisten.
2. Buatlah sebuah fungsi (stored function) untuk mendapatkan lama pengabdian/lama bekerja
(dalam tahun) seorang karyawan sejak direkrut (hired) hingga sekarang (current date).
Contoh: lama_kerja(10009) akan menghasilkan 37 (tahun).
Catatan: Apabila pegawai sudah tidak aktif bekerja (employee.to_date <> ‘9999-01-01’),
maka lama bekerja hanya sampai akhir masa aktifnya saja.
QUERY
```
DELIMITER //
CREATE FUNCTION lama_kerja (p_emp_no INT)
RETURNS INT
READS SQL DATA
DETERMINISTIC
BEGIN
DECLARE v_hire_date DATE;
DECLARE end_date DATE;
DECLARE years_worked DATE;
-- Ambil tanggal mulai kerja karyawan
SELECT hire_date INTO v_hire_date
FROM employees
WHERE emp_no = p_emp_no;
-- Jika karyawan tidak ditemukan, kembalikan 0 atau NULL (pilih 0
untuk simple)
IF v_hire_date IS NULL THEN
RETURN 0;
END IF;
-- Cari tanggal 'to_date' terakhir dari riwayat kerja (bisa dari
dept_emp atau titles)
-- Kita gunakan dept_emp sebagai indikator keaktifan
SELECT MAX(to_date) INTO years_worked
FROM dept_emp
WHERE emp_no = p_emp_no;
-- Tentukan tanggal akhir perhitungan:
-- Jika tidak pernah tercatat di dept_emp (seharusnya tidak
mungkin jika ada hire_date), anggap 0 tahun
IF years_worked IS NULL THEN
SET end_date = v_hire_date;
-- Jika to_date terakhir adalah '9999-01-01', berarti masih aktif,
gunakan tanggal hari ini
ELSEIF years_worked = '9999-01-01' THEN
SET end_date = CURDATE();
-- Jika to_date terakhir BUKAN '9999-01-01', berarti sudah tidak
aktif, gunakan tanggal itu
ELSE
SET end_date = years_worked;
END IF;
-- Hitung selisih tahun antara tanggal terakhir dan tanggal mulai
kerja
RETURN TIMESTAMPDIFF(YEAR, v_hire_date, end_date);
END //
DELIMITER ;
```
DESKRIPSI
Fungsi ini menerima emp_no sebagai input. Ia mengambil hire_date karyawan.
Kemudian mencari tanggal to_date paling akhir dari tabel dept_emp untuk
menentukan apakah karyawan masih aktif. Jika masih aktif (9999-01-01), tanggal
akhir dihitung sampai hari ini (CURDATE()); jika tidak, digunakan
tanggal to_date terakhir tersebut. Akhirnya, fungsi TIMESTAMPDIFF menghitung
selisih dalam tahun antara tanggal akhir dan hire_date, lalu mengembalikan hasilnya.)
Index
Buat 3 query dengan menggunakan kolom tertentu sebagai filter (yang belum ada indeksnya)
Jalankan query tersebut dan analisa dengan explain, buat tangkapan layarnya
Buat index pada kolom tersebut dengan menggunakan nama Anda sebagai nama indeksnya.
Jalankan ulang explain query tersebut, buat tangkapan layar hasilnya
1. Contoh 1
EXPLAIN
```
EXPLAIN SELECT replacement_cost
FROM film
WHERE replacement_cost > 20.00;
```
INDEX
```
CREATE INDEX muqoffin_cost ON film(replacement_cost);
```
2. Contoh 2
EXPLAIN
```
EXPLAIN SELECT rental_duration
FROM film
WHERE rental_duration = 6;
```
INDEX
```
CREATE INDEX muqoffin_rd ON film(rental_duration);
```
3. Contoh 3
EXPLAIN
```
EXPLAIN SELECT name
FROM language
WHERE name = 'English';
```
INDEX
```
CREATE INDEX muqoffin_ln ON language(name);
```
Procedure and Function
```
## Procedur
CREATE PROCEDURE `procedure_name`(
[IN|OUT|INOUT] `param_name` [DATATYPE] [(length)],
[IN|OUT|INOUT] `param_name2` [DATATYPE] [(length)],
...)
CONTAINS SQL
BEGIN
/* Your code goes here */
END;
------------------------------------------------------------
## Function
CREATE FUNCTION `function_name`(
[IN|OUT|INOUT] `param_name` [DATATYPE],
[IN|OUT|INOUT] `param_name2` [DATATYPE],
...
)
RETURNS [DATATYPE] [NOT] DETERMINISTIC
CONTAINS SQL
BEGIN
/* Your code goes here */
END;
```
1. Buatlah Procedure untuk menghitung jumlah film dengan kategori sesuai masukan.
Input : category_id
Output : jumlah film pada kategori yang dimasukkan
SQL
```
CREATE PROCEDURE `get_film_count_by_category` (
 IN p_category_id INT,
 OUT p_film_count INT
)
BEGIN
 SELECT COUNT(*) INTO p_film_count
 FROM film_category
 WHERE category_id = p_category_id;
END
```
CALL
```
## 2 Parameter IN(p_category_id) OUT(p_film_count)
CALL get_film_count_by_category(11, @count);
SELECT @count AS 'Number of Films';
```
2. Buatlah sebuah procedure yang menampilkan judul film dengan film_id sebagai masukan (input)
SQL
```
CREATE PROCEDURE `get_film_title_by_id` (
IN p_film_id INT
)
BEGIN
SELECT title
FROM film
WHERE film_id = p_film_id;
END
```
CALL
```
## 1 Parameter IN(p_film_id)
CALL get_film_title_by_id(11);
```
3. Buatlah fungsi untuk menghitung berapa kali setiap aktor bermain di film dengan masukan
(input) actor_id
SQL
```
CREATE PROCEDURE count_films_by_actor(
IN p_actor_id INT
)
BEGIN
SELECT COUNT(*) INTO film_count
FROM film_actor
WHERE actor_id = p_actor_id;
END
```
CALL
```
## 1 Parameter IN(p_actor_id)
CALL count_films_by_actor(11);
```
Trigger
```
CREATE TRIGGER `trigger_name`
[BEFORE|AFTER] [INSERT|UPDATE|DELETE] ON `nama_tabel` FOR EACH ROW
BEGIN
/* Your code goes here */
END;
```
1. Buatlah trigger yang mencatat perubahan harga sewa film setiap kali kolom rental_rate pada
tabel film diperbarui.
SQL
```
DELIMITER //
## buat terlebih dahulu tabel rental_rate log untuk menyimpan log
perubahan
CREATE TABLE IF NOT EXISTS rental_rate_log (
log_id INT AUTO_INCREMENT PRIMARY KEY,
film_id SMALLINT UNSIGNED,
old_rate DECIMAL(4,2),
new_rate DECIMAL(4,2),
update_date DATETIME,
user VARCHAR(50)
);
## buat triggernya
CREATE TRIGGER film_rental_rate_update
AFTER UPDATE ON film
FOR EACH ROW
BEGIN
IF OLD.rental_rate <> NEW.rental_rate THEN
INSERT INTO rental_rate_log (film_id, old_rate, new_rate,
update_date, user)
VALUES (NEW.film_id, OLD.rental_rate, NEW.rental_rate, NOW(),
CURRENT_USER());
END IF;
END //
DELIMITER ;
```
TRIGGER
```
## Pertama, lihat data sebelum diubah
SELECT film_id, title, rental_rate FROM film WHERE film_id = 1;
## Lakukan update harga sewa
UPDATE film SET rental_rate = 4.99 WHERE film_id = 1;
## Periksa apakah log tercatat
SELECT * FROM rental_rate_log WHERE film_id = 1;
```
2. Buatlah trigger yang secara otomatis mengisi kolom last_update di tabel customer ketika data
pelanggan diubah.
SQL
```
DELIMITER //
CREATE TRIGGER customer_update_time
BEFORE UPDATE ON customer
FOR EACH ROW
BEGIN
SET NEW.last_update = NOW();
END //
DELIMITER ;
```
TRIGGER
```
## Lihat nilai last_update sebelum diubah
SELECT customer_id, first_name, last_name, last_update FROM customer WHERE
customer_id = 1;
## Update data customer
UPDATE customer SET first_name = CONCAT(first_name, ' (updated)') WHERE
customer_id = 1;
## Periksa apakah last_update berubah
SELECT customer_id, first_name, last_name, last_update FROM customer WHERE
customer_id = 1;
```
3. Buatlah trigger yang mencatat perubahan data dari tabel actor.
SQL
```
DELIMITER //
## buat terlebih dahulu tabel actor_log log untuk menyimpan log
perubahan
CREATE TABLE IF NOT EXISTS actor_log (
log_id INT AUTO_INCREMENT PRIMARY KEY,
actor_id SMALLINT UNSIGNED,
old_first_name VARCHAR(45),
old_last_name VARCHAR(45),
new_first_name VARCHAR(45),
new_last_name VARCHAR(45),
change_date DATETIME,
action_type VARCHAR(10)
);
## buat triggernya
CREATE TRIGGER actor_after_update
AFTER UPDATE ON actor
FOR EACH ROW
BEGIN
INSERT INTO actor_log (actor_id, old_first_name, old_last_name,
new_first_name, new_last_name, change_date, action_type)
VALUES (NEW.actor_id, OLD.first_name, OLD.last_name, NEW.first_name,
NEW.last_name, NOW(), 'UPDATE');
END //
DELIMITER ;
```
TRIGGER
```
## Lihat data aktor sebelum diubah
SELECT * FROM actor WHERE actor_id = 1;
## Update data aktor
UPDATE actor SET first_name = CONCAT(first_name, ' (test)') WHERE actor_id
= 1;
## Periksa log yang dicatat
SELECT * FROM actor_log WHERE actor_id = 1;
```
4. Buatlah trigger yang mencatat tanggal saat penambahan data baru ke tabel inventory.
SQL
```
DELIMITER //
## buat terlebih dahulu tabel inventory_log untuk menyimpan log
perubahan
CREATE TABLE IF NOT EXISTS inventory_log (
log_id INT AUTO_INCREMENT PRIMARY KEY,
inventory_id MEDIUMINT UNSIGNED,
film_id SMALLINT UNSIGNED,
store_id TINYINT UNSIGNED,
creation_date DATETIME
);
## buat triggernya
CREATE TRIGGER inventory_after_insert
AFTER INSERT ON inventory
FOR EACH ROW
BEGIN
INSERT INTO inventory_log (inventory_id, film_id, store_id,
creation_date)
VALUES (NEW.inventory_id, NEW.film_id, NEW.store_id, NOW());
END //
DELIMITER ;
```
TRIGGER
```
## Lihat jumlah inventory sekarang
SELECT COUNT(*) FROM inventory;
## Tambahkan inventory baru
INSERT INTO inventory (film_id, store_id) VALUES (1, 1);
## Periksa log penambahan inventory
SELECT * FROM inventory_log ORDER BY log_id DESC LIMIT 1;
```
5. Buatlah trigger untuk mencatat perubahan pada kolom store_id di tabel staff.
SQL
```
DELIMITER //
## buat terlebih dahulu tabel staff_store_changes untuk menyimpan log
perubahan
CREATE TABLE IF NOT EXISTS staff_store_changes (
change_id INT AUTO_INCREMENT PRIMARY KEY,
staff_id TINYINT UNSIGNED,
old_store_id TINYINT UNSIGNED,
new_store_id TINYINT UNSIGNED,
change_date DATETIME
);
## buat triggernya
CREATE TRIGGER staff_store_update
AFTER UPDATE ON staff
FOR EACH ROW
BEGIN
IF OLD.store_id <> NEW.store_id THEN
INSERT INTO staff_store_changes (staff_id, old_store_id, new_store_id,
change_date)
VALUES (NEW.staff_id, OLD.store_id, NEW.store_id, NOW());
END IF;
END //
DELIMITER ;
```
TRIGGER
```
## Lihat data staff sebelum diubah
SELECT staff_id, first_name, last_name, store_id FROM staff WHERE staff_id
= 1;
## Update store_id staff
UPDATE staff SET store_id = IF(store_id = 1, 2, 1) WHERE staff_id = 1;
## Periksa catatan perubahan
SELECT * FROM staff_store_changes WHERE staff_id = 1;
```
6. Buatlah trigger yang mencatat perubahan data pada tabel category ke dalam tabel log bernama
category_log.
SQL
```
DELIMITER //
## buat terlebih dahulu tabel category_log untuk menyimpan log perubahan
CREATE TABLE IF NOT EXISTS category_log (
log_id INT AUTO_INCREMENT PRIMARY KEY,
category_id TINYINT UNSIGNED,
old_name VARCHAR(25),
new_name VARCHAR(25),
change_date DATETIME,
action_type VARCHAR(10)
);
## buat triggernya
CREATE TRIGGER category_after_update
AFTER UPDATE ON category
FOR EACH ROW
BEGIN
INSERT INTO category_log (category_id, old_name, new_name,
change_date, action_type)
VALUES (NEW.category_id, OLD.name, NEW.name, NOW(), 'UPDATE');
END //
DELIMITER ;
```
TRIGGER
```
## Lihat data kategori sebelum diubah
SELECT * FROM category WHERE category_id = 1;
## Update nama kategori
UPDATE category SET name = CONCAT(name, ' (updated)') WHERE category_id =
1;
## Periksa log perubahan
SELECT * FROM category_log WHERE category_id = 1;
```
7. Buatlah trigger yang mencegah perubahan data jika kolom amount pada tabel payment diubah
menjadi lebih dari $100. (Gunakan SIGNAL SQLSTATE '45000')
SQL
```
DELIMITER //
CREATE TRIGGER payment_amount_check
BEFORE UPDATE ON payment
FOR EACH ROW
BEGIN
IF NEW.amount > 100 THEN
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Pembayaran tidak dapat melebihi $100';
END IF;
END //
DELIMITER ;
```
TRIGGER
```
## Lihat data pembayaran
SELECT * FROM payment WHERE payment_id = 1;
## Coba update amount menjadi lebih dari $100 (akan error)
### Jalankan dengan hati-hati, karena akan menghasilkan error
UPDATE payment SET amount = 101.00 WHERE payment_id = 1;
## Coba update dengan nilai yang diizinkan
UPDATE payment SET amount = 5.99 WHERE payment_id = 1;
```
8. Buatlah trigger yang akan menyalin data lama dari baris customer ke tabel log bernama
customer_backup sebelum data di-update.
SQL
```
DELIMITER //
## buat terlebih dahulu tabel customer_backup untuk menyimpan log
perubahan/backup
CREATE TABLE IF NOT EXISTS customer_backup (
backup_id INT AUTO_INCREMENT PRIMARY KEY,
customer_id SMALLINT UNSIGNED,
store_id TINYINT UNSIGNED,
first_name VARCHAR(45),
last_name VARCHAR(45),
email VARCHAR(50),
address_id SMALLINT UNSIGNED,
active BOOLEAN,
create_date DATETIME,
backup_date DATETIME
);
## buat triggernya
CREATE TRIGGER customer_before_update
BEFORE UPDATE ON customer
FOR EACH ROW
BEGIN
INSERT INTO customer_backup (customer_id, store_id, first_name,
last_name, email, address_id, active, create_date, backup_date)
VALUES (OLD.customer_id, OLD.store_id, OLD.first_name, OLD.last_name,
OLD.email, OLD.address_id, OLD.active, OLD.create_date, NOW());
END //
DELIMITER ;
```
TRIGGER
```
## Update data customer
UPDATE customer SET email = CONCAT('updated_', email) WHERE customer_id =
1;
## Periksa backup yang dibuat
SELECT * FROM customer_backup WHERE customer_id = 1 ORDER BY backup_id
DESC LIMIT 1;
```
9. Buatlah trigger yang memunculkan pesan kesalahan jika pengguna mencoba menghapus film
yang masih tersedia di inventaris. (Gunakan SET MESSAGE_TEXT = 'pesan eror');
SQL
```
DELIMITER //
CREATE TRIGGER prevent_film_delete
BEFORE DELETE ON film
FOR EACH ROW
BEGIN
DECLARE inventory_count INT;
SELECT COUNT(*) INTO inventory_count
FROM inventory
WHERE film_id = OLD.film_id;
IF inventory_count > 0 THEN
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Film tidak dapat dihapus karena masih terdapat
dalam inventaris';
END IF;
END //
DELIMITER ;
```
TRIGGER
```
## Coba hapus film yang masih ada di inventaris (akan error)
### Jalankan dengan hati-hati, karena akan menghasilkan error
DELETE FROM film WHERE film_id = 1;
## Alternatif: Untuk menguji tanpa error, pilih film yang tidak ada di
inventaris
### Pertama, cari film yang tidak ada di inventaris
SELECT f.film_id, f.title
FROM film f
LEFT JOIN inventory i ON f.film_id = i.film_id
WHERE i.inventory_id IS NULL
LIMIT 1;
### Kemudian coba hapus film tersebut (sesuaikan film_id dengan hasil
query di atas)
DELETE FROM film WHERE film_id = [film_id yang tidak ada di inventaris];
```
10. Buatlah trigger untuk menghitung total pembayaran (total amount) setiap pelanggan saat data
baru ditambahkan ke tabel payment, dan simpan hasilnya di tabel customer_total.
SQL
```
DELIMITER //
CREATE TABLE IF NOT EXISTS customer_total (
customer_id SMALLINT UNSIGNED PRIMARY KEY,
total_payments DECIMAL(8,2) DEFAULT 0,
last_updated DATETIME
);
CREATE TRIGGER payment_after_insert
AFTER INSERT ON payment
FOR EACH ROW
BEGIN
INSERT INTO customer_total (customer_id, total_payments, last_updated)
VALUES (NEW.customer_id, NEW.amount, NOW())
ON DUPLICATE KEY UPDATE
total_payments = total_payments + NEW.amount, last_updated = NOW()
END //
DELIMITER ;
```
TRIGGER
```
## Tambahkan payment baru
INSERT INTO payment (customer_id, staff_id, rental_id, amount,
payment_date)
VALUES (1, 1, 1, 10.99, NOW());
## Periksa tabel customer_total
SELECT * FROM customer_total WHERE customer_id = 1;