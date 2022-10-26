# Makefile

all: repos.svg

repos.svg: repos.dot
	dot -Tsvg repos.dot > repos.svg

show: repos.svg
	eog repos.svg

install:
	install -m 644 repos.svg ../../2022-06-07-cuda-repos.svg


clean:
	rm -f repos.svg
