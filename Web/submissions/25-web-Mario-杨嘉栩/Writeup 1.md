# Writeup 1

## 1.[SWPUCTF 2021 新生赛]gift_F12

- 链接：https://www.nssctf.cn/problem/382
- 解题过程：F12 -> Element 找到flag 按照形式提交

## 2.robots.txt

- 链接：https://www.qsnctf.com/#/main/driving-range?page=1&category=&difficulty=&keyword=robots.txt&user_answer=&user_favorite=&tag_ids=
- 解题过程：题目提示正确配置robots.txt 于是在跳转链接末尾加上 /robots.txt 跳转页面得到文件名 修改为 /secret.php 得到账密取得flag<br>(其实没做出来。。。抓包F12dirsearch都试了)

## 3.[MoeCTF 2021]Web安全入门指北—GET

- 链接：https://www.nssctf.cn/problem/3412
- 解题过程：此题关键点在于```if ($moe == "flag") {
      echo $flag;
  }``` <br>
  此处比较运算符为 == 在PHP中为弱类型比较 会自动转换类型 因此尝试弱类型绕过如数组` ?moe[]=1 ` 然而此题是直接给出 `?参数=目标值`得到flag<br>
- 补充对URL结构的了解

![image-20260413223257822](C:\Users\CHNMa\AppData\Roaming\Typora\typora-user-images\image-20260413223257822.png)