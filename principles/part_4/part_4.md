# S.O.L.I.D

Les principe S.O.L.I.D sont des règles qui permettent d’augmenter la cohésion et de réduire le coupling, nous n’allons pas voir des implémentations en exemple (les cours précédent et celui sur les designs patterns sont suffisant pour appliquer ces règles)

## Principe de responsabilité unique (SRP) : 
Une classe ne devrait avoir qu'une seule raison de changer. Cela signifie que chaque classe doit se concentrer sur une tâche spécifique et ne pas avoir plusieurs responsabilités. L'avantage de suivre ce principe est que les classes deviennent plus cohérentes et plus faciles à comprendre, ce qui facilite la maintenance du code.

```py
class User:
    def __init__(self, name: str):
        self.name = name

    def get_user_name(self):
        return self.name

    def save_user(self, user):
        pass  # logique pour sauvegarder l'utilisateur dans une base de données
```

Ici, la classe User a deux responsabilités. Elle est responsable de la gestion de l'utilisateur et aussi de la sauvegarde de l'utilisateur dans la base de données. C'est une violation du Principe de Responsabilité Unique.

```py
class User:
    def __init__(self, name: str):
        self.name = name

    def get_user_name(self):
        return self.name

class UserDatabase:
    def save_user(self, user: User):
        pass  # logique pour sauvegarder l'utilisateur dans une base de données
```

Maintenant, la classe User est responsable uniquement de la gestion de l'utilisateur et la classe UserDatabase est responsable de la gestion de la base de données. Chacune a une seule responsabilité, ce qui respecte le Principe de Responsabilité Unique.

## Principe d'ouverture/fermeture (OCP)
 Les entités logicielles (classes, modules, etc.) devraient être ouvertes pour l'extension, mais fermées pour la modification. Cela signifie qu'il est possible d'ajouter de nouvelles fonctionnalités sans avoir à modifier le code existant. L'avantage de suivre ce principe est que les modifications de code sont minimisées, ce qui permet de limiter les risques de régressions et de bugs.

```py
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height


def total_area(rectangles):
    total = 0
    for rectangle in rectangles:
        total += rectangle.width * rectangle.height
    return total
```

Cette approche fonctionne bien pour les rectangles, mais suppose maintenant que nous voulions ajouter une autre forme, comme un cercle. Nous devrions modifier la fonction total_area pour gérer cette nouvelle forme, ce qui viole le principe OCP.

```py
from abc import ABC, abstractmethod
import math

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass


class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height


class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return math.pi * (self.radius ** 2)


def total_area(shapes):
    return sum(shape.area() for shape in shapes)
```

Maintenant, si nous voulons ajouter une nouvelle forme, nous n'avons qu'à créer une nouvelle classe qui hérite de Shape et à implémenter la méthode area. Il n'est pas nécessaire de modifier la fonction total_area, ce qui respecte le principe OCP.









## Principe de substitution de Liskov (LSP)
Les objets d'une classe dérivée doivent pouvoir être utilisés comme des objets de la classe de base sans que le comportement ne change. Cela signifie que les classes dérivées ne doivent pas avoir un comportement différent de celui de la classe de base, mais doivent plutôt étendre ou spécialiser son comportement. L'avantage de suivre ce principe est que cela facilite la réutilisation du code et rend les classes plus génériques.

```py
class Oiseau:
    def voler(self):
        return "Je peux voler!"

class Pingouin(Oiseau):
    def voler(self):
        return "Je ne peux pas voler!"

```

Dans cet exemple, le principe de Liskov est violé car si une fonction attend un Oiseau et que vous lui passez un Pingouin, la fonction s'attend à ce que l'oiseau puisse voler, mais cela ne sera pas le cas avec un Pingouin.

```py
class Oiseau:
    def voler(self):
        pass

class OiseauVolant(Oiseau):
    def voler(self):
        return "Je peux voler!"

class Pingouin(Oiseau):
    pass
```

Dans cette correction, nous avons introduit une nouvelle classe, OiseauVolant, qui hérite de la classe Oiseau. Cette classe représente spécifiquement les oiseaux qui peuvent voler. La classe Pingouin hérite toujours de la classe Oiseau, mais elle n'essaie pas de redéfinir la méthode voler. Ainsi, si une fonction attend un OiseauVolant, elle sait qu'elle obtiendra une instance qui peut voler. Si elle attend un Oiseau, elle sait qu'elle pourrait obtenir un OiseauVolant ou un Pingouin, et donc elle ne doit pas supposer que l'instance peut voler.

## Principe de ségrégation d'interface (ISP)

 Les interfaces doivent être spécifiques aux besoins des clients. Cela signifie que les interfaces doivent être divisées en plusieurs interfaces plus petites et spécialisées plutôt que de créer une seule interface qui satisfait tous les besoins des clients. L'avantage de suivre ce principe est que cela rend les interfaces plus cohérentes et plus faciles à utiliser pour les clients.

 ```py
from abc import ABC, abstractmethod

class MultiFunctionDevice(ABC):
    @abstractmethod
    def print(self, document): pass

    @abstractmethod
    def scan(self, document): pass

    @abstractmethod
    def fax(self, document): pass


class OldFashionedPrinter(MultiFunctionDevice):
    def print(self, document):
        # okay - print stuff
        pass

    def scan(self, document):
        raise NotImplementedError("Printer cannot scan!")

    def fax(self, document):
        raise NotImplementedError("Printer cannot fax!")

```

Dans cet exemple, OldFashionedPrinter est forcé d'implémenter scan et fax qui ne sont pas nécessaires, ce qui viole le principe ISP.

```py
class Printer:
    def print(self, document):
        pass


class Scanner:
    def scan(self, document):
        pass


class FaxMachine:
    def fax(self, document):
        pass


class MultiFunctionDevice(Printer, Scanner, FaxMachine):
    pass


class OldFashionedPrinter(Printer):
    def print(self, document):
        # okay - print stuff
        pass

```

Maintenant, OldFashionedPrinter n'est plus obligé d'implémenter les méthodes scan et fax qu'il n'utilise pas, ce qui est conforme au principe ISP.

Note : En python il n’y a pas d’interface à proprement parler, elles sont “équivalentes” aux classes abstraites (ou encore protocol) par conséquent ce principe ressemble grandement au principe de responsabilité unique (SRP)

## Principe d'inversion de dépendance (DIP)
Les modules de haut niveau ne doivent pas dépendre de modules de bas niveau, mais plutôt de leur abstractions. Cela signifie que les dépendances entre les classes doivent être inversées, de sorte que les classes de haut niveau dépendent des abstractions, plutôt que des classes de bas niveau. L'avantage de suivre ce principe est que cela permet de rendre le code plus modulaire, plus flexible et plus facile à tester.

- Les modules de haut niveau ne devraient pas dépendre des modules de bas niveau. Les deux devraient dépendre des abstractions.
- Les abstractions ne devraient pas dépendre des détails. Les détails devraient dépendre des abstractions.

```py
class MySQLDatabase:
    def save(self, data):
        # code pour sauvegarder des données dans une base de données MySQL

class DataProcessor:
    def __init__(self):
        self.database = MySQLDatabase()

    def process(self, data):
        # code pour traiter des données
        self.database.save(data)
```

Dans cet exemple, la classe DataProcessor dépend directement de la classe MySQLDatabase. Si nous décidons de changer de base de données (par exemple, passer à une base de données PostgreSQL), nous devons modifier la classe DataProcessor, ce qui viole le principe de l'ouverture/fermeture.

```py
from abc import ABC, abstractmethod

class Database(ABC):
    @abstractmethod
    def save(self, data):
        pass

class MySQLDatabase(Database):
    def save(self, data):
        # code pour sauvegarder des données dans une base de données MySQL

class PostgreSQLDatabase(Database):
    def save(self, data):
        # code pour sauvegarder des données dans une base de données PostgreSQL

class DataProcessor:
    def __init__(self, database: Database):
        self.database = database

    def process(self, data):
        # code pour traiter des données
        self.database.save(data)

```

Maintenant, DataProcessor dépend de l'abstraction Database, pas de l'implémentation spécifique. Nous pouvons facilement changer la base de données utilisée en passant une instance différente de Database à DataProcessor.

# S.T.U.P.I.D
Il y a aussi son contraire, ce qu’on appelle des anti-pattern à éviter à tout prix. S.T.U.P.I.D :

- Singleton : Le principe Singleton consiste à créer des objets qui ne peuvent être instanciés qu'une seule fois. Cette approche rend le code moins testable et moins flexible, car les objets Singleton sont souvent difficile à simuler ou à remplacer lors des tests unitaires.
- Tight Coupling : Le couplage étroit est une mauvaise pratique qui se produit lorsque les modules de code sont trop dépendants les uns des autres. Cela rend le code plus difficile à maintenir et à changer, car les modifications apportées à un module peuvent avoir des répercussions sur les autres modules qui en dépendent.
- Untestability : L'impénétrabilité est une mauvaise pratique qui se produit lorsque le code est difficile à tester. Cela se produit généralement lorsque le code est mal structuré ou que les dépendances sont mal gérées, ce qui rend les tests unitaires difficiles à écrire et à exécuter.
- Premature Optimization : L'optimisation prématurée est une mauvaise pratique qui se produit lorsqu'un développeur tente d'optimiser le code avant d'avoir identifié et résolu les problèmes de conception. Cela peut entraîner des modifications inutiles du code, ce qui le rend plus complexe et plus difficile à maintenir.
- Indescriptive Naming : La désignation indescriptible est une mauvaise pratique qui consiste à utiliser des noms de variables, de fonctions et de classes qui ne sont pas explicites ou qui ne décrivent pas clairement leur fonction ou leur objectif. Cela peut rendre le code plus difficile à comprendre et à maintenir, en particulier pour les nouveaux développeurs qui ne sont pas familiers avec le code existant.
- Duplication : La duplication est une mauvaise pratique qui se produit lorsqu'un développeur copie et colle du code d'un endroit à un autre plutôt que de créer une fonction ou une classe qui peut être réutilisée. Cela peut entraîner des problèmes de cohérence et de maintenabilité, car des modifications doivent être apportées à plusieurs endroits pour maintenir le code synchronisé.
