extract:
	pybabel extract -F babel.cfg  -o messages.pot .

update: extract
	pybabel update -i messages.pot -d translations

compile:
	pybabel compile -d translations

