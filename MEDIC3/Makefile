medic_presentation.html: Makefile medic_presentation.md styles.html imgs/*
	pandoc -t revealjs -s -o medic_presentation.html medic_presentation.md \
		-V revealjs-url=https://unpkg.com/reveal.js --include-in-header=styles.html \
		-V theme=black -V transition=fade -V slideNumber=true -V mathjax=true --slide-level=3 \
		-f markdown-tex_math_dollars-raw_tex --template=revealjs.template

all: medic_presentation.html

clean:
	rm -f medic_presentation.html
