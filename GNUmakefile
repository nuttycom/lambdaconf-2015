SOURCE = slides.md
STYLE = style.html
THEME = night

PANDOC = pandoc

all: postprocess

clean:
	rm -f index.html

postprocess: reveal
	sed -i'' -e 's/reveal.min/reveal/' index.html
	sed -i'' -e "s/simple.css/$(THEME).css/" index.html

reveal: $(SOURCE) $(STYLE)
	$(PANDOC) -f markdown --smart -t revealjs -V theme=$(THEME) --include-in-header=$(STYLE) -s $(SOURCE) -o index.html
