SOURCE = slides.md
STYLE = style.txt

PANDOC = pandoc

all: reveal

clean:
	rm -f index.html

reveal: $(SOURCE) $(STYLE)
	$(PANDOC) -f markdown --smart -t revealjs -V theme=night --include-in-header=$(STYLE) -s $(SOURCE) -o index.html
