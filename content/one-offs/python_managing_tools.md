---
layout: default
title: python managing tools
---

{: .title}
Python managing tools

* TOC
{: toc}

# Préface
Dans les deux cas, `poetry` et `pipenv`, il y des `lockfile` en plus de leurs fichiers d'installation respectifs `pyproject.toml` et `Pipfile`.
Je n'ai pas encore compris l'avantage des `lockfile` par rapport aux fichiers d'installation.

Un mot sur l'import des modules locaux : l'idée est que les modules locaux, même s'ils sont ajoutés aux environnements virtuels,
ne sont pas copiés à l'intérieur de l'environnement. C'est à dire que les modifications sur le module local seront
automatiquement et immédiatement (sans passer par publication / pull) utilisables par les modules qui l'importe.

# `poetry` sucks.
## Installation
Pas d'installation globale à part avec `pip`.
L'installation locale choisit le premier parmi les exécutables `python` et `python3`. S'il trouve `python` et que c'est une version 2,
les commandes `poetry` utiliseront `python2`. Notamment :
- les initialisations des environnements seront on `python2.7`,
- toute commande `poetry` sort un superbe warning disant que cette version de `python` ne sait pas faire de threads.

J'ai hacké en virant la recherche de l'exécutable `python` pour le forcer à faire de `python3` (et ça marche).

## Utiliser un environnement
Dans un dossier `hello/` :

	poetry init -n

crée un `pyproject.toml` de base, puis on accède à l'environnement avec :

	poetry shell

## Ajouter des modules
### Modules globaux

	poetry add keras

### Modules locaux
Il faut faire un dossier qui contient :
- un fichier `pyproject.toml` dont le champ `name` contient le nom du module `python`,
- le dossier contenant le module `python`.

Notamment, le dossier qu'on ajoute **ne peut pas être** le module `python`. Après ça :

	poetry add /path/to/folder

(pas le chemin vers le module ! parce qu'il faut qu'il trouve `pyproject.toml`).

## Recréer un environnement
Se mettre dans un dossier avec le `pyproject.toml` correct, puis :

	poetry install

# `pipenv` sucks.
## Installation

	apt-get install pipenv

et ça, c'est bien.

## Utiliser un environnement
Dans un dossier `hello/` :

	pipenv shell

va créer un fichier `Pipfile` et va activer l'environnement.

## Ajouter des modules
### Modules globaux

	pipenv install keras

### Module locaux
Il faut faire un dossier qui contient un `setup.py`. Notamment, ça ne marche pas à coup de `Pipfile`, ce qui est bien pourri. Après ça :

	pipenv install -e /path/to/folder

## Recréer un environnement
Se mettre dans un dossier avec le `Pipfile` correct, puis :

	pipenv install

# The winner sucks.
Mon problème de base était de pouvoir importer des modules sans en faire de copie. Les deux font ça, mais `poetry` le fait d'une manière
plus homogène : un dossier `poetry` importe un autre dossier `poetry`, alors que `pipenv` oblige un `setup.py` du plus mauvais effet. Yay ?

En clair, j'ai galéré avec poetry mais l'unique chose que je voulais est gérée correctement. Tout est venu plus facilement
avec pipenv mais l'unique chose que je voulais est géré n'importe comment. Yay ?
