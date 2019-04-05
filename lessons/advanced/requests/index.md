# Requests + Click

## Co je to internet

[shrnutí pro začátečníky]({{ lesson_url('fast-track/http') }})


## Requests

Knihovna určená pro HTTP požadavky na straně klienta. Poskytuje mnohem
pohodlnější rozhraní než standardní knihovna Python.

Instalace ve virtuálním prostředí:

```console
$ python -m pip install requests
```

První ochutnávka
```pycon
>>> import requests
>>> response = requests.get("http://example.com/")
>>> response
<Response [200]>
```

Vyzkoušejte různé adresy:
* `https://httpbin.org/get`
* `https://httpbin.org/redirect-to?url=http://example.com&status_code=301`

Podívejte se na tyto atributy:

* `text`
* `status_code`
* `encoding`
* `history`

a tuto metodu:

* `json()`

Co asi znamenají?

## Parametry pro GET

```pycon
>>> requests.get("https://httpbin.org/redirect-to?url=http://example.com&status_code=301")
>>> params = {"status_code": 301, "url": "https://example.com"}
>>> r = requests.get("https://httpbin.org/redirect-to", params=params)
```

## Posílání dat

```pycon
>>> r = requests.post("https://httpbin.org/post", {"ahoj": "svete"})
```

## Stažení velkého souboru

```python
# surprise.py
import requests
with requests.get("https://placekitten.com/400/600") as r:
    r.raise_for_status()
    with open("kitten.jpg", "w") as f:
        for chunk in r.iter_content(8196):
            if chunk:
                f.write(chunk)
```

## Kurzovní lístek

`http://www.cnb.cz/cs/financni_trhy/devizovy_trh/kurzy_devizoveho_trhu/denni_kurz.txt`

Parametr `data` ve formátu `DD.MM.YYYY`.


```python
# cnb.cz
def parse_rates(data):
    header, names, *lines = data.splitlines()
    rates = {}
    for line in lines:
        _, _, amount, currency, value = line.replace(",", ".").split("|")
        rates[currency] = float(value) / float(amount)
    return rates
```

Napište funkci `get_exchange_rates(date=None)`


# Click

Existuje hodně knihoven, které umožňují zpracovávat argumenty na příkazové řádce. Jenom samotná standardní knihovna Pythonu má `getopt`, `optparse` a `argparse`. Ty ale nejsou úplně příjemné na používání.

`Click` je jednoduchý.

## Instalace

```console
$ python -m pip install click colorama
```

## Hello world

```python
# hello.py

import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
def hello(count, name):
    for x in range(count):
        click.echo('Hello %s!' % name)

if __name__ == '__main__':
    hello()
```

Rozdíl mezi argumentem a přepínačem: jeden je volitelný, druhý ne.

Typy přepínačů:
 * `click.INT`
 * `click.FLOAT`
 * `click.FILE`


```console
$ python cnb.py --help
Usage: cnb.py [OPTIONS] AMOUNT

Options:
  --date TEXT
  --to TEXT
  --help       Show this message and exit.
```


```python
# cnb.cz
import click
import requests


def parse_rates(data):
    header, names, *lines = data.splitlines()
    rates = {}
    for line in lines:
        _, _, amount, currency, value = line.replace(",", ".").split("|")
        rates[currency] = float(value) / float(amount)
    return rates


def get_exchange_rates(date=None):
    query = {}
    if date:
        query["date"] = date
    response = requests.get("http://www.cnb.cz/cs/financni_trhy/devizovy_trh/kurzy_devizoveho_trhu/denni  _kurz.txt", params=query)
    return parse_rates(response.text)


@click.command()
@click.option("--date")
@click.option("--to", multiple=True)
@click.argument("amount", type=click.FLOAT)
def cnb(amount, date=None, to=None):
    rates = get_exchange_rates(date)
    for currency in sorted(rates):
        if not to or currency in to:
            converted = amount / rates[currency]
            click.echo(f"{amount} CZK = {converted} {currency}")


if __name__ == "__main__":
    cnb()
```
