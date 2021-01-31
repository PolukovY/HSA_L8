##setup


SET autocommit=0;
SET GLOBAL innodb_status_output=ON;
SET GLOBAL innodb_status_output_locks=ON;

##init

run init.sql

drop table accounts;
create table if not exists accounts
(
    id int unsigned auto_increment
        primary key,
    login varchar(255) not null,
    balance bigint default 0 not null,
    created_at timestamp default now()
) collate=utf8mb4_unicode_ci;

insert into accounts (login, balance) values ('petya', 1000);
insert into accounts (login, balance) values ('vasya', 2000);
insert into accounts (login, balance) values ('mark', 500);


#Read uncommitted

## T1

start transaction ;

select * from accounts;

insert into accounts (login, balance, created_at) values  ('ivan', 750, now());
delete from accounts where login = 'vasya';

update accounts set balance = 1500 where login = 'petya';

rollback ;

## T2

start transaction ;

select * from accounts;

select sum(balance) from accounts;

T2 see uncommitted data and could do wrong logic. After T1 rollback T2 sum will be changed.

#Read committed

##T1

start transaction ;

select * from accounts;

insert into accounts (login, balance, created_at) values  ('ivan', 750, now());
delete from accounts where login = 'vasya';

update accounts set balance = 1500 where login = 'petya';

select * from accounts;

select sum(balance) from accounts;

commit;

##T2

start transaction ;

select * from accounts;

select sum(balance) from accounts;

T2 doesn't see uncommitted data, after commit T2 see new data. T1 sees his local data.

#Repeatable read

##T1

start transaction ;

select * from accounts;

insert into accounts (login, balance, created_at) values  ('ivan', 750, now());
update accounts set balance = 1500 where login = 'vasya';
delete from accounts where login = 'petya';

select * from accounts;

select sum(balance) from accounts;

commit;

##T2

start transaction ;

select * from accounts;

update accounts set balance = 2000 where login = 'vasya';
select sum(balance) from accounts;

commit ;

T2 will wait until T2 commits or rollback changes.

Mysql doesn't have reading phantoms


Article to readhttps://habr.com/ru/post/469415/ & https://ru.wikipedia.org/wiki/%D0%A3%D1%80%D0%BE%D0%B2%D0%B5%D0%BD%D1%8C_%D0%B8%D0%B7%D0%BE%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D0%BE%D1%81%D1%82%D0%B8_%D1%82%D1%80%D0%B0%D0%BD%D0%B7%D0%B0%D0%BA%D1%86%D0%B8%D0%B9
