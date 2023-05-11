Tests

[projet github repo](https://github.com/JonathanNdambaPro/S.E)

## Tests unitaires

les tests unitaires sont la plus petite unité de test ici, on va chercher à voir si une fonction exécute bien ce qu’on lui demande sans intervention extérieure (libraire, module, etc.). Maintenant que nous avons un exemple.

[code](https://github.com/JonathanNdambaPro/S.E/blob/main/package_1/file_1.py)

```py
def file_1_function(sentence_1: str = "Hello", sentence_2: str = "World") -> str:
    """Concatenante two sentences

    Parameters
    ----------

    sentence_1: str
        first of sentence to concatenate

    sentence_2: str
        second of sentence to concatenate

    Returns
    -------
    str
        sentence concatenate
    """
    return f"{sentence_1} {sentence_2}"
```

Nous tester cette fonction, pour ce faire, nous allons coder un test unitaire.

Vous devez créer un dossier qui comporte le prefix “test” ici, il s’appelle test, mais il pourrait s’appeler test\_folder\_1, cette règle est équivalente pour les fichiers python (test\_file\_1).

Pytest va chercher tous les fichier et folder commençant par “test”.

![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.002.png)

Maintenant codons une fonction qui lance la fonction et vérifie si le résultat rendu est bien au format qu’on attend

Tips :

Avant toute chose, nous vous conseillons de créer un fichier pytest.ini à la racine du projet avec les lignes suivantes

[code](https://github.com/JonathanNdambaPro/S.E/blob/main/pytest.ini)

```ini
[pytest]
pythonpath = .
norecursedirs = lib bin include lib64
```
pythonpath : Indique à pytest où regarder avant de lancer les tests norecursedirs : indique quels dossiers où ne pas regarder

sans ça pytest pourrait se montrer très capricieux.

Pensez à remplir le Makefile avec la nouvelle commande pytest :

[code](https://github.com/JonathanNdambaPro/S.E/blob/main/Makefile)

```Makefile
.PHONY: test
test:
	pytest --cov=package_2/sub_package --cov-report term-missing --cov-fail-under=80
```
Contenu de test\_file\_1 :

[code](https://github.com/JonathanNdambaPro/S.E/blob/main/test/test%20unitaire/test_file_1.py)

```py
from package_1 import file_1


def test_file_1_function():
    value = file_1.file_1_function()
    assert "Hello World" == value
```
lors de ce test, on appelle la fonction file\_1\_function()

```py
def file_1_function(sentence_1: str = "Hello", sentence_2: str = "World") -> str:
    """Concatenante two sentences

    Parameters
    ----------

    sentence_1: str
        first of sentence to concatenate

    sentence_2: str
        second of sentence to concatenate

    Returns
    -------
    str
        sentence concatenate
    """
    return f"{sentence_1} {sentence_2}"
```

Comme nous n'avons pas mis de valeur, la sortie attendu est “Hello World”,

assert “Hello Wolrd” == value pourrait être compris comme “vérifie que file\_1\_function() renvoie bien la valeur ‘Hello world’ ”

```base
make test
```

![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.007.png)

maintenons-nous allons faire rater le test volontairement

```py
from package_1 import file_1


def test_file_1_function():
    value = file_1.file_1_function()
    assert "Goodbye World" == value
```

![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.009.png)

le test parle de lui-même, il a essayé de comparer Goodbye World et Hello World qui ne sont pas la même chose, corrigeons-le.

```py
from package_1 import file_1


def test_file_1_function():
    value = file_1.file_1_function(sentence_1="Goodbye")
    assert "Goodbye World" == value
```
![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.011.png)

Maintenant plus compliqué

[code](https://github.com/JonathanNdambaPro/S.E/blob/main/package_2/sub_package/file_4.py)

```py
import typing as t
from pathlib import Path

import pandas as pd

from package_2.file_3 import file_3_function


def file_4_function():
    file_3_function()
    print("package_2.file_4_function")


def read_header_csv(path: Path) -> t.List:
    if path.is_file() and path.suffix == ".csv":
        print(path)
        df_data = pd.read_csv(path)
        columns = [column for column in df_data.columns]
        return columns

    columns = []
    return columns
```

nous allons tester uniquement read\_file, cette fonction va lire un fichier et sortir sont contenus.


Cependant, il y a un problème, les librairies pathlib et pandas sont utilisées et les tests unitaires doivent être isolé, si on laisse ce test comme ça nous ne testons pas exclusivement le comportement que nous avons implémenté, mais également pathlib et pandas ce qui casse la logique du tests unitaire.

Pour passer corriger ce comportement, nous devons utiliser un mock :

- Un "mock" (ou "mock object") est un objet simulé qui est utilisé pour remplacer un objet réel dans le cadre de tests unitaires.

Regardons l’implémentation ensemble.

- D’abord installation de pytest-mock : 

```bash
pip install pytest-mock
pip install pandas
```

[code où remplacer](https://github.com/JonathanNdambaPro/S.E/blob/main/test/test%20unitaire/test_file_4.py)
```py
from package_2.sub_package import file_4
from dataclasses import dataclass, field
import typing as t

@dataclass
class PathFaker:
  suffix: str "csv"
  def is_file(self):
    return True

def generate_fake_columns():
  return ["column_1", "column_2", "column_3"]

@dataclass
class DataFrameFaker:
  columns: t.List = field(default_factory-generate_fake_columns)

def test_file_4_read_file(mocker):
  faker_dataframe DataFrameFaker ()
  mocker.patch("package_2.sub_package.file_4.pd.read_csv", return_value=faker_dataframe)
  project_root_path = PathFaker()
  value = file_4.read_header_csv (project_root_path)
  assert value == ["column_1", "column_2", "column_3"]
```
Examinons chaque étape :

- PathFaker :
  - is\_file : doit retourner vrai si le fichier existe, nous allons supposer que c’est vrai en retournant True
  - suffix : retourne l’extension d’un fichier, la fonction read\_header\_csv suppose que nous devons avoir un csv
- DataFrameFaker :
  - on suppose cet objet à un header (colonne d’un fichier csv) et que l’on peut y accéder par l’attribut columns
- mocker : le mocker va permettre d’accéder à une fonction/méthode dans le code et “d’en prendre le contrôle”, au lieu de laisser cette fonction comme elle le ferai normalement, on va lui indiquer ce qu’elle retourne

![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.015.png)

Cette façon n’est pas optimale dans le sens ou les fakers sont dans le même fichier que les tests unitaires, l’idée est de regrouper tous les fakers dans un seul fichier pour que ça soit plus propre et que l’on puisse les réutiliser (DRY).

Pour ça il faut créer un fichier conftest à la racine du dossier test

![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.016.png)

[code](https://github.com/JonathanNdambaPro/S.E/blob/main/test/conftest.py)
```py
import typing as t
from dataclasses import dataclass, field

import pytest


@dataclass
class PathFaker:
    suffix: str = ".csv"
    state: bool = True

    def is_file(self):
        return self.state


def generate_fake_columns():
    return ["column_1", "column_2", "column_3"]


@dataclass
class DataFrameFaker:
    columns: t.List = field(default_factory=generate_fake_columns)


@pytest.fixture()
def path_faker():
    path_faker = PathFaker()
    return path_faker


@pytest.fixture()
def path_faker_false():
    path_faker = PathFaker(state=False)
    return path_faker


@pytest.fixture()
def dataframe_faker():
    dataframe_faker = DataFrameFaker()
    return dataframe_faker
```
Dans le conftest, nous avons réuni tous les fakers dans un seul fichier et nous avons ajouté une chose, les fixtures.

- fixtures : les fixtures vont permettre de réutiliser les sorties de fonctions
- conftest.py : permet de stocker les fixtures pour qu’elles soient utilisables dans n’importe quel tests

```py
import typing as t

from package_2.sub_package import file_4


def test_file_4_read_file(mocker, path_faker, dataframe_faker):
    mocker.patch("package_2.sub_package.file_4.pd.read_csv", return_value=dataframe_faker)
    value = file_4.read_header_csv(path_faker)

    assert value == ["column_1", "column_2", "column_3"]
```

On peut voir que le test est beaucoup plus propre.

Modifions légèrement le Makefile mais avant installons pytest-cov

```bash
pip install pytest-cov
```

```MAkefile
.PHONY: docs
docs:
	pdoc --html --skip-errors --force --output-dir docs package_1

.PHONY: test
test:
	pytest --cov=package_2/sub_package --cov-report term-missing --cov-fail-under=80
```
![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.020.png)

On regarde la couverture du code. L'objectif de la couverture de code est de s'assurer que le code est bien testé et que tous les chemins possibles sont couverts. En effet, même si un test unitaire réussit, il est possible que certaines parties du code ne soient jamais exécutées, ce qui peut causer des erreurs imprévues dans d'autres parties du programme.

On voit que le file\_4.py à une couverture 71%, on peut voir également les lignes manquantes

[code](https://github.com/JonathanNdambaPro/S.E/blob/main/package_2/sub_package/file_4.py)
```py
import typing as t
from pathlib import Path

import pandas as pd

from package_2.file_3 import file_3_function


def file_4_function():
    file_3_function()
    print("package_2.file_4_function")


def read_header_csv(path: Path) -> t.List:
    if path.is_file() and path.suffix == ".csv":
        print(path)
        df_data = pd.read_csv(path)
        columns = [column for column in df_data.columns]
        return columns

    columns = []
    return columns
```

les lignes 20 et 21 ne sont pas testés, il faudra donc essayer de faire un deuxième test qui passera n’ira pas dans la boucle if. De manière générale, il faut tester tous les chemins possibles de la fonction, c'est ce qu’on appelle la complexité cyclomatique.

“La complexité cyclomatique est une mesure de la complexité d'un programme informatique qui permet d'évaluer le nombre de chemins différents qui peuvent être empruntés dans le code source. Plus la complexité cyclomatique est élevée, plus le code est difficile à comprendre, à maintenir et à tester. En effet, une complexité cyclomatique élevée signifie qu'il y a plus de chemins possibles dans le code, ce qui peut rendre plus difficile la création de tests unitaires exhaustifs et la détection des erreurs de code.”

On revient donc au principe S.O.L.I.D (surtout le S) KISS et de cohésion forte et coupling faible, vos fonctions doivent être simples et ne pas prendre trop de responsabilité, si votre code est trop dur à tester alors, il faudrait surement diviser voiture en fonction en plusieurs sous fonction.

[code](https://github.com/JonathanNdambaPro/S.E/blob/main/test/conftest.py)
```py
import typing as t
from dataclasses import dataclass, field

import pytest


@dataclass
class PathFaker:
    suffix: str = ".csv"
    state: bool = True

    def is_file(self):
        return self.state


def generate_fake_columns():
    return ["column_1", "column_2", "column_3"]


@dataclass
class DataFrameFaker:
    columns: t.List = field(default_factory=generate_fake_columns)


@pytest.fixture()
def path_faker():
    path_faker = PathFaker()
    return path_faker


@pytest.fixture()
def path_faker_false():
    path_faker = PathFaker(state=False)
    return path_faker


@pytest.fixture()
def dataframe_faker():
    dataframe_faker = DataFrameFaker()
    return dataframe_faker
```

on a ajouté la fixture path\_faker\_false pour que is\_file retourne false

```py
import typing as t

from package_2.sub_package import file_4


def test_file_4_read_file(mocker, path_faker, dataframe_faker):
    mocker.patch("package_2.sub_package.file_4.pd.read_csv", return_value=dataframe_faker)
    value = file_4.read_header_csv(path_faker)

    assert value == ["column_1", "column_2", "column_3"]


def test_file_4_read_file_false(mocker, path_faker_false, dataframe_faker):
    mocker.patch("package_2.sub_package.file_4.pd.read_csv", return_value=dataframe_faker)
    value = file_4.read_header_csv(path_faker_false)

    assert value == []
```
![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.023.png)

## Tests d’intégrations

Comme nous avons vu les tests unitaires se concentre exclusivement sur ce que nous avons codé, nous devons mocker les comportements d’api (librairies/module) externes et les données qui devrait interagir avec notre code.

Les tests d’intégrations eux vont laisser les interactions entre les modules/librairies/api externe/données avec le code que nous avons implémenté pour voir si tout “s’intègre” bien les uns avec les autres.

```py
import typing as t
from pathlib import Path

import pandas as pd

from package_2.file_3 import file_3_function


def file_4_function():
    file_3_function()
    print("package_2.file_4_function")


def read_header_csv(path: Path) -> t.List:
    if path.is_file() and path.suffix == ".csv":
        print(path)
        df_data = pd.read_csv(path)
        columns = [column for column in df_data.columns]
        return columns

    columns = []
    return columns
```

D'abord, nous allons mieux intégrer les tests.

![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.024.png)

Maintenant l’implémentation du test.

```py
from pathlib import Path

from package_2.sub_package import file_4


def test_file_4_read_file():
    header_csv = "column_1,column_2,column_3"
    current_folder = Path(".").resolve()
    path_csv_path = current_folder / "test.csv"

    with path_csv_path.open("w") as path:
        path.write(header_csv)

    value = file_4.read_header_csv(path_csv_path)

    assert value == ["column_1", "column_2", "column_3"]

    path_csv_path.unlink()
```
- La première partie du code, nous allons créer un header column\_1,column\_2,column\_3
- Dans un deuxième temps, on récupère le fichier csv précédemment créé.
- On compare le header et on le compare à [column\_1, column\_2, column\_3]
- Pour finir, on supprime le fichier test.csv

![](Aspose.Words.c4511e00-d8df-449c-a725-310484ec3acb.026.png)

## Tests fonctionnels

les tests fonctionnels sont des tests qui ne sont pas sous votre responsabilité, une personne va tester vos nouvelles fonctionnalités et vérifier que votre code fonctionne bien comme on s’attend.
