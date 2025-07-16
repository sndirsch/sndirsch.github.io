# Makefile

all: repos.svg

repos.svg: repos.dot
	dot -Tsvg repos.dot > repos.svg

show: repos.svg
	eog repos.svg

install:
	install -m 644 repos.svg ../../2025-07-16-gfx-repo.svg


clean:
	rm -f repos.svg

