# Templates Method

Le pattern templates méthode ne vas pas changer la méthode de la classe mais ces composant.

## Sans pattern
[bitcoin.py](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/templates_method/before/bitcoin.py)

```py
class BitcoinTradingBot:
    def buy(self, amount: int) -> None:
        print(f"Buying {amount} BTC satoshi.")

    def sell(self, amount: int) -> None:
        print(f"Selling {amount} BTC satoshi.")

    def should_buy(self, prices: list[int]) -> bool:
        return prices[-1] > prices[-2]

    def should_sell(self, prices: list[int]) -> bool:
        return prices[-1] < prices[-2]

    def get_price_data(self) -> list[int]:
        return [
            35_842_00,
            34_069_00,
            33_871_00,
            34_209_00,
            32_917_00,
            33_931_00,
            33_370_00,
            34_445_00,
            32_901_00,
            33_013_00,
        ]

    def get_amount(self) -> int:
        return 20000

    def trade(self) -> None:
        prices = self.get_price_data()
        amount = self.get_amount()

        if self.should_buy(prices):
            self.buy(amount)
        if self.should_sell(prices):
            self.sell(amount)
```

[ethereum.py](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/templates_method/before/ethereum.py)

```py

class EthereumTradingBot:
    def buy(self, amount: int) -> None:
        print(f"Buying {amount/10000000:.7f} Ethereum.")

    def sell(self, amount: int) -> None:
        print(f"Selling {amount/10000000:.7f} Ethereum.")

    def should_buy(self, prices: list[int]) -> bool:
        return prices[-1] > prices[-2]

    def should_sell(self, prices: list[int]) -> bool:
        return prices[-1] < prices[-2]

    def get_price_data(self) -> list[int]:
        return [
            2_381_00,
            2_233_00,
            2_300_00,
            2_342_00,
            2_137_00,
            2_156_00,
            2_103_00,
            2_165_00,
            2_028_00,
            2_004_00,
        ]

    def get_amount(self) -> int:
        return 10000

    def trade(self) -> None:
        prices = self.get_price_data()
        amount = self.get_amount()

        if self.should_buy(prices):
            self.buy(amount)
        if self.should_sell(prices):
            self.sell(amount)
```

[main.py](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/templates_method/templates_before/main.py)

```py
from bitcoin import BitcoinTradingBot
from ethereum import EthereumTradingBot


def main():
    bitcoin_trader = BitcoinTradingBot()
    bitcoin_trader.trade()

    ethereum_trader = EthereumTradingBot()
    ethereum_trader.trade()


if __name__ == "__main__":
    main()
```
Ici nos deux bots sont rassemblant, surtout la méthode trade qui est similaire pour les deux, pour rendre le code plus générique, nous allons faire une classe abstraite mère des deux classes.


## Avec pattern

[trading_bot.py](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/templates_method/classic/trading_bot.py)

```py
from abc import ABC, abstractmethod


class TradingBot(ABC):
    @abstractmethod
    def buy(self, amount: int) -> None:
        """Buy the given amount."""

    @abstractmethod
    def sell(self, amount: int) -> None:
        """Sell the given amount."""

    @abstractmethod
    def should_buy(self, prices: list[int]) -> bool:
        """Should the bot buy?"""

    @abstractmethod
    def should_sell(self, prices: list[int]) -> bool:
        """Should the bot sell?"""

    @abstractmethod
    def get_price_data(self) -> list[int]:
        """Get the price data."""

    @abstractmethod
    def get_amount(self) -> int:
        """Get the amount to trade."""

    def trade(self) -> None:
        prices = self.get_price_data()
        amount = self.get_amount()

        if self.should_buy(prices):
            self.buy(amount)
        if self.should_sell(prices):
            self.sell(amount)
```

[bitcoin.py](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/templates_method/classic/bitcoin.py)

```py
from trading_bot import TradingBot


class BitcoinTradingBot(TradingBot):
    def buy(self, amount: int) -> None:
        print(f"Buying {amount} BTC satoshi.")

    def sell(self, amount: int) -> None:
        print(f"Selling {amount} BTC satoshi.")

    def should_buy(self, prices: list[int]) -> bool:
        return prices[-1] > prices[-2]

    def should_sell(self, prices: list[int]) -> bool:
        return prices[-1] < prices[-2]

    def get_price_data(self) -> list[int]:
        return [
            35_842_00,
            34_069_00,
            33_871_00,
            34_209_00,
            32_917_00,
            33_931_00,
            33_370_00,
            34_445_00,
            32_901_00,
            33_013_00,
        ]

    def get_amount(self) -> int:
        return 10
```

[ethereum.py](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/templates_method/classic/ethereum.py)

```py
from trading_bot import TradingBot


class EthereumTradingBot(TradingBot):
    def buy(self, amount: int) -> None:
        print(f"Buying {amount/10000000:.7f} Ethereum.")

    def sell(self, amount: int) -> None:
        print(f"Selling {amount/10000000:.7f} Ethereum.")

    def should_buy(self, prices: list[int]) -> bool:
        return prices[-1] > prices[-2]

    def should_sell(self, prices: list[int]) -> bool:
        return prices[-1] < prices[-2]

    def get_price_data(self) -> list[int]:
        return [
            2_381_00,
            2_233_00,
            2_300_00,
            2_342_00,
            2_137_00,
            2_156_00,
            2_103_00,
            2_165_00,
            2_028_00,
            2_004_00,
        ]

    def get_amount(self) -> int:
        return 100000
```

[main.py](https://github.com/JonathanNdambaPro/code_adanced_course/blob/main/design%20patterns/templates_method/classic/main.py)

```py
from bitcoin import BitcoinTradingBot
from ethereum import EthereumTradingBot


def main():
    bitcoin_trader = BitcoinTradingBot()
    bitcoin_trader.trade()

    ethereum_trader = EthereumTradingBot()
    ethereum_trader.trade()


if __name__ == "__main__":
    main()
```
Concentrons-nous sur la classe abstraite TradingBot

cette classe abstraite sert d’interface pour ses classes filles, pour rappel une interface sert à indiquer aux classes filles quelles méthodes doivent être implémenté, c’est une signature ou “blueprint”. Mais elle n’indique pas comment ces méthodes doivent être implémenté, ce qui laisse la liberté aux classes filles les actions des méthodes.

Comme nos classes filles, EtheureumTradingBot et BitcoinTradingBot ont les mêmes méthodes, mais des comportements différents, TradingBot va juste indiquer qui faut les implémenter à l’exception d’une méthode trade.

Au lieu de coder la méthode trade dans chacune des classes filles, on va se l’implémenté dans la classe mère pour qu’elles puissent en hérité et ainsi ne pas créer de redondance.

En faites le template method pattern, c'est du polymorphisme amélioré on va implémenter directement les méthode qui se comporte pareil directement dans la classe mère pour ne pas à avoir à le coder partout.
