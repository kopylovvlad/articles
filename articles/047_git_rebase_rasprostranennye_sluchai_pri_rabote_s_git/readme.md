# Git rebase: распространенные случаи при работе с git

![Photo by Ross Sneddon on Unsplash](image01.jpeg)

Каждый разработчик работает с git, но к сожалению не все используют rebase в своей работе. Мало кто знает в каких рабочих кейсах он может пригодится, поэтому я написал данную статью с разборами рабочих ситуаций.

**Предупреждение**: Все примеры представленные здесь про изменение над одним файлом. Это удобно чтобы показать процесс работы, но увеличивает вероятность что при rebase возникнут конфликты. Поэтому разбор конфликтов в статье не будет.

Не удивляйтесь что после rebase придется пушить код в репозиторий через параметр `-f`. Это нужно чтобы принудительно переписать изменения в репозитории

```
git push origin head -f
```

Но есть более безопасный способ запушить код - использовать ключ [--force-with-lease](https://stackoverflow.com/questions/8939977/git-push-rejected-after-feature-branch-rebase)

```
git push origin head --force-with-lease
```

## Кейсы

1. Поправить коммит в середине (fixup + rebase)
1. Удалить коммит в середине (rebase + drop)
1. Объединить коммиты и поменять текст итогового коммита (rebase/soft reset)
1. Поправить последний коммит (soft reset)
1. Поправить текст последнего коммита (amend)
1. Поменять коммиты местами (rebase)

![Photo by Wonderlane on Unsplash](image02.jpeg)

## 1) Задача: поправить коммит в середине

**Решение**: сделать fixup

У нас есть 4 коммита в которых мы правили один файл

```
$ git log --oneline master..HEAD
aa871c7 (HEAD -> example_1, base_branch) add third paragraph to readme
3774350 add second paragraph to readme
2143008 add first paragraph to readme
1da47ab add title to readme

$ git diff master..HEAD
diff — git a/README.md b/README.md
index e69de29..315b9d6 100644
 — — a/README.md
+++ b/README.md
@@ -0,0 +1,7 @@
+## Hello
+
+First paragraph
+
+Second paragraph
+
+Third paragraph
```

Нам нужно поправить коммит который добавляет первый параграф (2143008). Открываем файл README.md и правим текст первого параграфа.

```
$ git diff
diff — git a/README.md b/README.md
index 315b9d6..14dc0b7 100644
 — — a/README.md
+++ b/README.md
@@ -1,7 +1,3 @@
 ## Hello
-First paragraph
-
-Second paragraph
-
-Third paragraph
+First paragraph :)
```

Так как мы в примере меняем один файл, то нужно сделать так каким он должен выглядеть после коммита (2143008) т.е там должен быть только заголовок и первый параграф.

Делаем "fixup коммит" задача которого поправить другой коммит в истории. Синтаксис операции следующий `git commit — fixup <commit_id>`

```
$ git add README.md
$ git commit — fixup 2143008
[example_1 4d0edd6] fixup! add first paragraph to readme
 1 file changed, 1 insertion(+), 5 deletions(-)
```

Готово, у нас есть новый коммит, но надо еще приложить усилия чтобы он применился к коммиту (2143008).

```
$ git log --oneline master..HEAD
4d0edd6 (HEAD -> example_1) fixup! add first paragraph to readme
aa871c7 (base_branch) add third paragraph to readme
3774350 add second paragraph to readme
2143008 add first paragraph to readme
1da47ab add title to readme
```

Для этого нужно сделать rebase к коммиту на 1 ниже того который мы хотим поправить (1da47ab) (возможно здесь придется в явном виде решить конфликты, если git не понял как связать hotfix).

```
$ EDITOR=vim git rebase -i --autosquash 1da47ab
# во время выполнения вы увидите что коммит fixup встал на свое “правильное” место, а не в конце истории
1 pick 2143008 add first paragraph to readme
2 fixup 4d0edd6 fixup! add first paragraph to readme
3 pick 3774350 add second paragraph to readme
4 pick aa871c7 add third paragraph to readme
Successfully rebased and updated refs/heads/example_1.
```

Посмотрим теперь на историю.

```
$ git log --oneline master..HEAD
8b8e8b2 (HEAD -> example_1) add third paragraph to readme
9dc3b2f add second paragraph to readme
557541f add first paragraph to readme
1da47ab add title to readme
```

И на изменения.

```
$ git diff master..HEAD
diff — git a/README.md b/README.md
index e69de29..ff9fc5e 100644
 — — a/README.md
+++ b/README.md
@@ -0,0 +1,7 @@
+## Hello
+
+First paragraph :)
+
+Second paragraph
+
+Third paragraph
```

Ура, у нас получилось!

![Photo by Xavi Cabrera on Unsplash](image03.jpeg)

## 2) Задача: удалить коммит в середине

**Решение**: сделать drop

У нас есть те же самые 4 коммита где мы правили один файл

```
$ git log --oneline master..HEAD
aa871c7 (HEAD -> example_2, base_branch) add third paragraph to readme
3774350 add second paragraph to readme
2143008 add first paragraph to readme
1da47ab add title to readme

$ git diff master..HEAD
diff — git a/README.md b/README.md
index e69de29..315b9d6 100644
 — — a/README.md
+++ b/README.md
@@ -0,0 +1,7 @@
+## Hello
+
+First paragraph
+
+Second paragraph
+
+Third paragraph
```

Мы поняли что первый параграф нам не нужен. Удалять изменения новым коммитом не хочется т.к. мы может просто удалить старый. Для этого делаем rebase к интересующему коммиту (1da47ab).

```
$ EDITOR=vim git rebase -i 1da47ab
```

И прописываем drop к тому коммиту, который мы хотим удалить.

```
1 drop 2143008 add first paragraph to readme
2 pick 3774350 add second paragraph to readme
3 pick aa871c7 add third paragraph to readme
```

Посмотрим что у нас получилось.

```
$ git log --oneline master..HEAD
2b87851 (HEAD -> example_2) add third paragraph to readme
db792c4 add second paragraph to readme
1da47ab add title to readme

$ git diff master..HEAD
diff — git a/README.md b/README.md
index e69de29..3354aa2 100644
 — — a/README.md
+++ b/README.md
@@ -0,0 +1,5 @@
+## Hello
+
+Second paragraph
+
+Third paragraph
```

![Photo by Bianca Ackermann on Unsplash](image04.jpeg)

## 3) Задача: объединить коммиты и поменять текст итогового коммита

**Решение**: сделать rebase

У нас есть один README.md файл и несколько "wip" коммитов.

```
$ git log --oneline master..HEAD
33e58db (HEAD -> example_3) wip
585c26b wip
173ab1f wip
c1f8895 wip
85855c0 wip
$ git diff master..HEAD
diff — git a/README.md b/README.md
index e69de29..6222dc3 100644
 — — a/README.md
+++ b/README.md
@@ -0,0 +1,7 @@
+## h2 header
+
+Some text. Bla-bla-bla
+
+### h3 header
+
+Some text again. LOL
```

Наша задача объединить это коммиты в один с осмысленным текстом. Для этого надо сделать rebase на 5 коммитов назад.

```
$ git log --oneline master..HEAD | wc -l
5
$ EDITOR=vim git rebase -i HEAD~5
```

И поменять pick на squash

```
1 pick 85855c0 wip
2 squash c1f8895 wip
3 squash 173ab1f wip
4 squash 585c26b wip
5 squash 33e58db wip
```

После этого вручную поменять сообщение первого коммита.

```
1 # This is a combination of 5 commits.
2 # This is the 1st commit message:
3
4 Create README.md file
5
6 # This is the commit message #2:
7
8 wip
9
10 # This is the commit message #3:
11
12 wip
13
14 # This is the commit message #4:
15
16 wip
17
18 # This is the commit message #5:
19
20 wip
```

Вроде получилось

```
[detached HEAD 30310eb] Create README.md file
 Date: Sun Dec 6 22:00:41 2020 +0300
 1 file changed, 7 insertions(+)
Successfully rebased and updated refs/heads/example_3.
```

Посмотрим что у нас есть в итоге.

```
$ git log --oneline master..HEAD
30310eb (HEAD -> example_3) Create README.md file
$ git diff master..HEAD
diff — git a/README.md b/README.md
index e69de29..6222dc3 100644
 — — a/README.md
+++ b/README.md
@@ -0,0 +1,7 @@
+## h2 header
+
+Some text. Bla-bla-bla
+
+### h3 header
+
+Some text again. LOL
```

Ура, данные объединились в один коммит.

![Photo by Fabian Kühne on Unsplash](image05.jpeg)


**Решение 2**: сделать soft reset

Делаем soft reset на 5 коммитов назад

```
$ git log --oneline master..HEAD | wc -l
5
$ git reset --soft HEAD~5
```

И сохранить изменения в один новый коммит.

```
$ git ci -m "Add README.md file"
[example_3.2 22513de] Add README.md file
 1 file changed, 7 insertions(+)
```

Посмотрим что получилось.

```
$ git log --oneline master..HEAD
22513de (HEAD -> example_3.2) Add README.md file

$ git diff master..HEAD
diff — git a/README.md b/README.md
index e69de29..6222dc3 100644
 — — a/README.md
+++ b/README.md
@@ -0,0 +1,7 @@
+## h2 header
+
+Some text. Bla-bla-bla
+
+### h3 header
+
+Some text again. LOL
```

Все получилось!

![Photo by Delaney Van on Unsplash](image06.jpeg)

## 4) Задача: поправить последний коммит

**Решение**: сделать soft reset

Итак, у нас есть два коммита. По каким-то причинам нам надо поправить последний коммит.

```
$ git log --oneline master..HEAD
4efcc59 (HEAD -> example_3.2) Add more text to README.md
22513de Add README.md file
```

Можно сделать fixup, но он требует несколько действий. Гораздо быстрее сделать по-другому - убираем последний коммит, но оставляем все текущие изменения

```
$ git reset --soft HEAD~1
```

В итоге у нас в истории остался только предыдущий коммит.

```
$ git log --oneline master..HEAD
22513de (HEAD -> example_3.2) Add README.md file
```

Но все последние изменения остались на файловой системе

```
$ git st
On branch example_3.2
Changes to be committed:
 (use "git reset HEAD <file>…" to unstage)
modified: README.md
```

Теперь мы можем поменять данные в файле которые нам надо и создать новый коммит.

```
$ git ci -a -m "Add more text to README.md"
[example_3.2 87eaff0] Add more text to README.md
 1 file changed, 2 insertions(+)
```

Вуаля, все получилось

```
$ git log --oneline master..HEAD
87eaff0 (HEAD -> example_3.2) Add more text to README.md
22513de Add README.md file
```

![Photo by Lydia Tallent on Unsplash](image07.jpeg)

## 5) Задача: поправить текст последнего коммита

**Решение**: сделать amend

Иногда в тексте коммита можно допустить ошибку и его надо переписать. Самый простой способ это сделать amend.

```
$ git log --oneline master..HEAD
fb77ed2 (HEAD -> example_3.2) improve documentattion
87eaff0 Add more text to README.md
22513de Add README.md file
```

В последнем коммите опечатка в слове “documentation”. Поправим его

```
$ git commit --amend -m "improve documentation"
[example_3.2 ed02c8a] improve documentation
 Date: Sun Dec 6 22:57:22 2020 +0300
 1 file changed, 2 insertions(+)

git log --oneline master..HEAD
ed02c8a (HEAD -> example_3.2) improve documentation
87eaff0 Add more text to README.md
22513de Add README.md file
```

![Photo by Mor THIAM on Unsplash](image08.jpeg)

## 6) Задача: поменять коммиты местами

**Решение**: сделать rebase

Итак, у нас опять есть три коммита

```
$ git log --oneline master..HEAD
98bfce0 (HEAD -> example_6) add footer to Readme.md
9fd509c add header to Readme.md
589ad4d add text to Readme.md
```

В которых мы меняли один файл.

```
$ git diff master..HEAD
diff --git a/README.md b/README.md
index e69de29..2c6d4f9 100644
 — — a/README.md
+++ b/README.md
@@ -0,0 +1,7 @@
+# Header
+
+Text. Bla-bla-bla.
+Some text.
+And text, again.
+
+# Footer
```

Как поменять местами коммиты чтобы сперва в истории было добавление футера (98bfce0), а потом уже хедера (9fd509c)? Для этого делаем интерактивный rebase на 3 коммита назад

```
$ EDITOR=vim git rebase -i HEAD~3
1 pick 589ad4d add text to Readme.md
2 pick 9fd509c add header to Readme.md
3 pick 98bfce0 add footer to Readme.md
```

И меняем расположение коммитов

```
1 pick 589ad4d add text to Readme.md
2 pick 98bfce0 add footer to Readme.md
3 pick 9fd509c add header to Readme.md
```

Посмотрим что получилось.

```
$ git diff master..HEAD
diff — git a/README.md b/README.md
index e69de29..2c6d4f9 100644
 — — a/README.md
+++ b/README.md
@@ -0,0 +1,7 @@
+# Header
+
+Text. Bla-bla-bla.
+Some text.
+And text, again.
+
+# Footer

$ git log --oneline master..HEAD
a4e641f (HEAD -> example_6) add header to Readme.md
cf87d0a add footer to Readme.md
589ad4d add text to Readme.md
```

Давайте попробуем откатиться назад чтобы проверить что уберется хедер, а футер останется.

```
$ git reset --hard HEAD~1
HEAD is now at cf87d0a add footer to Readme.md

$ cat README.md

Text. Bla-bla-bla.
Some text.
And text, again.
# Footer
```

И взглянем на историю коммитов.

```
$ git log --oneline master..HEAD
cf87d0a (HEAD -> example_6) add footer to Readme.md
589ad4d add text to Readme.md
```

Вот и все! Надеюсь что данная статья была полезная 😁

[Medium](https://kopilov-vlad.medium.com/git-rebase-%D1%80%D0%B0%D1%81%D0%BF%D1%80%D0%BE%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5-%D1%81%D0%BB%D1%83%D1%87%D0%B0%D0%B8-%D0%BF%D1%80%D0%B8-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B5-%D1%81-git-f5f9d3bc8047)
