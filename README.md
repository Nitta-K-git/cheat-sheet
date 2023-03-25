# cheat-sheet
個人的なメモ書きをまとめたもの．各項目の詳細はそれぞれのリンク先を参照

## Install

```sh
pyenv install 3.10.4
poetry env use 3.10.4
poetry install
```

## Build document

```sh
# build and start viewer
poetry run sphinx-autobuild docs/source docs

# build only
poetry run sphinx-build -b html docs/source/ docs
```
