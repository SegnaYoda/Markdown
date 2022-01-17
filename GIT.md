Любые проекты, включающие в себя больше одного участника, требуют персонализации, и GIT - не исключение. Для корректной работы репозитория, а также для того, чтобы иметь возможность установить автора внесенных изменений, необходимо представиться - ввести имя пользователя и электронную почту.
```
$ git config --global user.name "Noname Incognito"
$ git config --global user.email innoname@email.com
```
Отныне все ваши действия в GIT будут фиксироваться под вашим именем.
Чтобы узнать текущие настройки (например, под какими данными мы авторизованы), можно запросить эту  информацию у GIT:

`$ git config --list`
Данная команда выдаст все конфигурации, установленные для конкретного пользователя. Также есть возможность узнать конкретную настройку, например, ввести такую команду:
`$ git config user.name`
которая выдаст информацию строго по запросу.
Менять имя пользователя и емейл в процессе работы **крайне нежелательно**, поскольку могут возникнуть ошибки, связанные с множественностью авторов, и отследить историю изменений будет проблематично.  

Также **git config** позволяет изменить визуальный вывод данных для пользователя - настройки интерфейса могут облегчить восприятие, выделив нужные места разными цветами:

`$ git config --global color.ui true`

Для создания __нового__ проекта в GIT необходимо:
- перейти в директорию, где будет располагаться проект, и, вызвав командную строку, ввести команду
`$ git init`
- С её помощью создаётся поддиректория __.git__, содержащая всю базу Git-репозитория, проект ещё не находится под контролем Git,
для добавления файлов в Git необходимо добавить их в индекс и произвести коммит, используя команды **git add** и **git commit**:
`$ git add .$ git commit -m 'initial project version'`
- Все  файлы, находящиеся в директории, будут закоммичены. С этого момента проект находится под версионным контролем, поскольку в Git-репозитории располагаются отслеживаемые файлы и начальный коммит.
***
Для клонирования существующего репозитория:
-необходимо скопировать с сервера слепок всех файлов, имеющих отношение к репозиторию.  Для этого используется команда **git clone**, к которой добавляется ссылка на существующий репозиторий, а также (при  необходимости) новое имя склонированного репозитория:
`$ git clone https://github.com/libgit2/libgit2`
- в данном случае репозиторий будет называться libgit2
Возможно также задать другое имя директории:
`﻿$ git clone https://github.com/libgit2/libgit2 newrepo`
- данном случае репозиторий будет называться newrepo


Для GIT файлы делятся на неотслеживаемые, то есть не находящиеся под версионным контролем (например, файлы в новом репозитории до самого первого коммита) и отслеживаемые, к которым относятся все файлы, попавшие под контроль.

Отслеживаемые файлы могут находиться в трёх состояниях:  неизменённом, изменённом и подготовленном к коммиту. Для определения состояния файлов вызывается команда **git status**.

В **неизменённом** состоянии файлы находятся сразу после коммита, но до того, как были внесены новые изменения - то есть, когда файлы в рабочей директории соответствуют файлам, попавшим в последний коммит.

Вывод git status в таком состоянии будет выглядеть так:
```
$ git status
On branch master
nothing to commit, working directory clean
```
Файлы переходят в **изменённое** состояние после того, как в них вносятся любые изменения, но до того, как они будут подготовлены к коммиту. Изменениями являются добавление новых файлов, изменение старых, переименование, удаление - любые действия, воздействующие на содержимое репозитория.

Так, например, чтобы создать файл README (внести изменение в репозиторий), **необходимо воспользоваться командой touch**:

`$ touch README`
Вывод состояния git status будет отображать все произведенные изменения, при этом предлагая действия по подготовке изменений к коммиту:
```
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

nothing added to commit but untracked files present (use "git add" to track)
```
Указание  "**untracked files**" означает, что внесенные изменения ещё не зафиксированы и могут быть утеряны, если не будут добавлены к коммиту.
Это очень важный момент, поскольку не все появившиеся/измененные файлы действительно необходимо отправлять в коммит - если какие-то файлы, отмеченные как  **Untracked**, нам не нужны, достаточно просто не предпринимать никаких действий в их отношении, чтобы они не попали в следующий коммит.

Если же мы хотим, чтобы в коммит попали нужные нам изменения, эти изменения необходимо добавить под версионный контроль, то есть, начать отслеживать и, тем самым, подготовить к коммиту.

**Подготовленным к коммиту** (staged) файл становится, если после всех произошедших с ним изменений он **индексируется** (т.е. вносится во временное хранилище) с помощью команды **git add**:

`$ git add README`

В таком случае вывод статуса git status будет иметь следующий вид:
```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
```
Таким же образом индексируются измененные файлы - команда **git add** универсальна для добавления к коммиту любых изменений.
Один и тот же файл может находиться одновременно в двух разных состояниях (быть одновременно подготовленным и не подготовленным к коммиту). Это происходит, если изменения в файле были проиндексированы, однако после этого в тот же файл были внесены другие изменения - в таком случае, вывод команды **git status** будет выглядеть так:
```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    modified:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   README
```
Если коммит будет произведен сейчас, то изменения, внесенные после индексации, будут утеряны. Чтобы этого избежать, необходимо снова проиндексировать измененные файлы.
***
***Просмотр изменений***

Порой необходимо выяснить, какие именно изменения были внесены в файлы - для этого команды **git status**, только называющей измененные файлы, недостаточно. Команда **git diff** решает эту проблему, указывая, какие именно строки были изъяты/добавлены. Отслеживаться с помощью **git diff** могут только те файлы, которые были добавлены под версионный контроль (проиндексированы). ﻿

Команда **git diff** отвечает на несколько вопросов, например, какие именно изменения не были проиндексированы **(git diff)** и какие именно проиндексированные изменения войдут в следующий коммит **(git diff --staged или git diff --cached)**.

***А теперь - слайды***
У нас есть файлы README, который был создан и проиндексирован после последнего коммита, и **Example.md**, который был изменён после последнего коммита, но ещё не проиндексирован.

Для того, чтобы проследить судьбу **Example.md** и выяснить, какие изменения были внесены, но не были проиндексированы, необходимо вызвать команду **git diff**, в таком случае её вывод будет таким:
```
$ git diff
diff --git a/Example.md b/Example.md
index 643e24f..87f08c8 100644
--- a/Example.md
+++ b/Example.md
@@ -119,3 +119,4 @@ at the
 ## Starter Projects

 See our [projects list](https://github.com/libgit2/libgit2/blob/development/PROJECTS.md).
+# test line
```
где  *diff --git a/Example.md b/Example.md* - показывает, какие именно файлы сравниваются, где **а** - версия после индексирования, **b** - текущая версия,
*--- a/Example.md* - дословно читается как "из версии **а** удален файл Example.md",
*+++ b/Example.md* - дословно читается как "в версию **b** добавлен файл Example.md"

Вывод данной команды покажет разницу между содержимым индекса и содержимым рабочего каталога, то есть, от последнего индексирования до текущего момента.

Если необходимо выяснить, какие изменения произошли с *README* с момента последнего коммита до последней индексации, необходимо вызвать команду  **git diff --staged** (равноценной заменой ей будет команда **git diff --cached**).

В таком случае вывод этой команды будет выглядеть так:
```
$ git diff --staged
diff --git a/README b/README
new file mode 100644
index 0000000..03902a1
--- /dev/null
+++ b/README
@@ -0,0 +1 @@
+My Project
```
где  *diff --git a/README b/README* - показывает, какие именно файлы сравниваются, где **а** - версия последнего коммита, **b** - версия последнего индексирования,
*--- /dev/null* - "удалено отсутствие файла", то есть, в предыдущем коммите данного файла не было,
*+++ b/README* - в версию **b** добавлен файл README.
________________________________________________________
В случае, если проиндексированный файл *Example2.md* существовал в последнем коммите, (т.е. *git status* называет его не *new file*, а *modified*), вывод команды **git diff --staged** будет таким:
```
$ git diff --staged
diff --git a/Example2.md b/Example2.md
index 8ebb991..643e24f 100644
--- a/Example2.md
+++ b/Example2.md
@@ -65,7 +65,8 @@ branch directly, things can get messy.
 Please include a nice description of your changes when you submit your PR;
 if we have to read the whole diff to figure out why you're contributing
 in the first place, you're less likely to get feedback and have your change
-merged in.
+merged in. Also, split your changes into comprehensive chunks if you patch is
+longer than a dozen lines.

 If you are starting to work on a particular area, feel free to submit a PR
 that highlights your work in progress (and note in the PR title that it's)
```
где  *diff --git a/Example2.md b/Example2.md* - показывает, что сравниваются файлы *Example2* версии **а** (версии последнего коммита) и версии **b** (версии последнего индексирования)
**--- a/Example2.md** - из версии **а** извлечён файл *Example2.md* (т.е. версия этого файла из последнего коммита),
*+++ b/Example2.md* - в версию **b** добавлен файл  *Example2.md* (т.е. новая версия этого файла, содержащаяся в последнем индексировании).

Также, если при выводе **git status** файл указывается одновременно как подготовленный и не подготовленный к коммиту (см. предыдущий шаг), то есть, если файл содержит и проиндексированные, и непроиндексированные изменения вместе, то возможно просмотреть и те, и другие изменения - для этого необходимо использовать и **git diff**, и **git diff --staged (--cached)** в зависимости от того, какие именно изменения нужно просмотреть.
***
***Игнорирование***

Порой игнорировать что-то незначительное - единственный способ сосредоточиться на чём-то действительно важном. К GIT это применимо в полной мере - вместе с нужными файлами, которые мы собирались зафиксировать, в коммит может попасть "мусор" - например, автоматически генерируемые файлы.

Для того, чтобы лишнее не попало ни в категорию неотслеживаемых, ни в следующий коммит, необходимо создать файл *.gitignore*, в котором будет перечислено то, что будет отсеиваться автоматически:
```
$ cat .gitignore
/tmp/
!/tmp/no-cache-data/
/.idea/
/temp/*
*.pyc
*.[oa]
/t?mp/*
*~
```
Например, данное содержимое файла означает, что GIT будет игнорировать файлы, заканчивающиеся на **.о** или на **.а**, а также на ~ (тильду), которой в некоторых текстовых редакторах обозначаются временные файлы.

Для формирования файла  *.gitignore*, являющегося списком исключений из поля зрения GIT, необходимо соблюдать следующие правила:

1. используются стандартные glob-шаблоны - упрощенные регулярные выражения:
- символ *(*)* подразумевает любое количество символов,
- последовательность *[abc]* - обозначает любой символ из перечисленных в скобках, то есть либо a, либо b, либо c,
- символ *(?)* подразумевает один символ,
- последовательность *[a-z]* - обозначает любой символ из указанного в скобках интервала, то есть все от a до z,
2. комментарии в файле .gitignore можно ввести, начав строку с символа #,
3. (/) слэш в начале и конце строки укажет на конкретный каталог,
4. (!) восклицательный знак в начале строки инвертирует шаблон: будут проигнорированы все файлы из указанного каталога, кроме тех, что отмечены восклицательным знаком,
5. (/temp/*) - игнорировать все файлы директории temp, но не игнорировать подпапки
***
***Коммит***

Один из важнейших элементов работы с GIT, как и работы в целом - закрепление успеха. В случае GIT идет речь о фиксации изменения, достигаемой путём введения команды **git commit**.

Правилом хорошего кода является обязательный комментарий к каждому коммиту, который может быть введен как одновременно с командой **(git commit -m "What`s a great commit!")**, так и в текстовом редакторе, который автоматически откроется при выполнении команды **git commit**﻿ без заданного комментария.

В первом случае результат будет выглядеть подобным образом:
```
$ git commit -m "What`s a great commit!"
[master 453bc5d] What`s a great commit!
3 files changed, 3 insertions(+)
create mode 100644 README
```
Во втором случае текстовый редактор (например, Vim), предложит ввести комментарий:
```
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
# new file:   README
# modified:   Example.md
#

modified:   Example2.md
#
~
~
~
".git/COMMIT_EDITMSG" 9L, 283C
```
Способы добавления комментария равнозначны - всё зависит от того, какой вариант удобнее пользователю.

Данные, которые не были проиндексированы, не добавляются в коммит и остаются в рабочем каталоге.