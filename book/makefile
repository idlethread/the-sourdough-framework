# Macros for commands
LATEX := latexmk -cd -pdf -pdflatex="pdflatex -interaction=nonstopmode" -use-make
EBOOK := tex4ebook -d epub -c tex4ebook.cfg
CLEAN := latexmk -cd -c -use-make
EBOOK_CONVERT := kindlegen
CHECK_1 := lacheck
CHECK_2 := chktex
WEBSITE_DIR := static_website_html

ifdef DEBUG
EBOOK += -a debug
endif

# List all files that are dependencies
chapters = baking basics bread-types flour-types history intro\
		non-wheat-sourdough sourdough-starter storing-bread troubleshooting\
		wheat-sourdough

src_tables  := $(wildcard tables/table-*.tex)
src_figures := $(wildcard figures/fig-*.tex)

src_tex := $(foreach directory, $(chapters), $(wildcard $(directory)/*.tex))
src_tex += book.tex book_sans_serif.tex references.bib figures/vars.tex
src_tex += supporters.csv

images := $(wildcard images/*/*.jpg)
images += $(wildcard images/*/*.png)
images += $(foreach directory, $(chapters), $(wildcard $(directory)/*.jpg))
images += $(foreach directory, $(chapters), $(wildcard $(directory)/*.png))

src_all := $(src_tex) $(src_figures) $(src_tables) tex4ebook.cfg book.mk4 $(images)

tgt_figures := $(patsubst %.tex, %.png,$(src_figures))

# Default rules for pdf and ebooks, getting overwritten when built in a
# sub-directory
%.pdf: %.tex
	$(LATEX) $<

# TODO: check if it works on github CI
%.png: %.tex
	@echo "\input{./vars}" > $@.in
	# \b is backspace..
	@echo "\\\begin{document}" >> $@.in
	@cat $< >> $@.in
	@echo "\end{document}" >> $@.in
	$(LATEX) $@.in

book_serif/book.pdf: $(src_all)
	$(LATEX) -output-directory=book_serif book.tex

book_sans_serif/book_sans_serif.pdf: $(src_all)
	$(LATEX) -output-directory=book_sans_serif book_sans_serif.tex

epub/%.epub: %.tex $(src_all)
	$(EBOOK) -f epub $<

epub/%.mobi: epub/%.epub
	$(EBOOK_CONVERT) $< -o $(notdir $@)

epub/%.azw3:  epub/%.epub
	$(EBOOK_CONVERT) $< -o $(notdir $@)

# Now with the rules
# Expected usual rules first
.PHONY: default
default: build_serif_pdf

.PHONY: all
all: bake

.PHONY: help
help:
	@echo ""
	@echo "default: builds the book in pdf format (serif)"
	@echo ""
	@echo "all: pdf and ebooks serif and sans-serif accessible version, same as"
	@echo "     build release"
	@echo "bake: same as build all"
	@echo ""
	@echo "check: runs static analysis checker on LaTeX source to spot"
	@echo "       programming or typographic mistakes"
	@echo ""
	@echo "clean: delete all intermediate files keep targets (pdf/ebook)"
	@echo "mrproper: delete all generated files intermediate and pdf/ebooks"
	@echo "clean_figures: delete intermediate TikZ files"
	@echo ""
	@echo "build_ebook: builds only the ebook serif and accessible version"
	@echo "build_pdf: builds both serif and accessible pdf"
	@echo ""
	@echo "build_sans_serif_ebook: build accessible ebook only"
	@echo "build_sans_serif_pdf: build accessible pdf only"
	@echo ""
	@echo "build_serif_ebook: build serif ebook only"
	@echo "build_serif_pdf: build serif pdf only"
	@echo ""
	@echo "figures: build TikZ figures"
	@echo ""
	@echo "release_serif: build serif only version of pdf and ebooks"
	@echo "release_sans_serif: build sans-serif/accessible version of pdf and ebooks"
	@echo ""
	@echo "Debug targets:"
	@echo ""
	@echo "show_tools_version: Show version of tools used on the build machine"
	@echo ""
	@echo "print-X: print makefile variable X"
	@echo ""
	@echo "printvars: print all variables in the makefile"
	@echo ""
	@echo "set DEBUG i.e make DEBUG=1 build_ebook to add debug flags to commands"


# Finally project specif targets
.PHONY: build_pdf
build_pdf: build_serif_pdf build_sans_serif_pdf

.PHONY: build_serif_pdf
build_serif_pdf: book_serif/book.pdf

.PHONY: build_sans_serif_pdf
build_sans_serif_pdf: book_sans_serif/book_sans_serif.pdf

.PHONY: build_ebook
build_ebook: build_serif_ebook build_sans_serif_ebook

.PHONY: build_serif_ebook
build_serif_ebook: epub/book.epub epub/book.mobi epub/book.azw3 | make_release_dir

.PHONY: build_sans_serif_ebook
build_sans_serif_ebook: epub/book_sans_serif.epub epub/book_sans_serif.mobi \
						epub/book_sans_serif.azw3 | make_release_dir

.PHONY: export_figures
# Requires that you have docker running on your computer.
export_figures: build_pdf $(tgt_figures)
	cd figures/ && bash export_figures.sh

# Goal is not really to have 0 warning reported but we should check we don't
# add many and if we do, we know they are false positive
PHONY: check
check: $(SRC_TEX)
	@echo "Running: " $(CHECK_1)
	$(CHECK_1) book.tex
	@echo ""
	@echo "Running: " $(CHECK_2)
	$(CHECK_2) book.tex

.PHONY: clean_figures
clean_figures:
	- $(CLEAN) $(patsubst %.tex, %.png.in, $(src_figures))
	- rm $(patsubst %.tex, %.png.pdf, $(src_figures))
	- rm $(patsubst %.tex, %.png.in, $(src_figures))

.PHONY: clean_ebook_build
clean_ebook_build:
	-rm  book*.aux
	-rm  book*.run.xml
	-rm  book*.bcf
	-rm  book*.blg
	-rm  book*.log
	-rm  book*.4tc
	-rm  book*.4ct
	-rm  book*.dvi
	-rm  book.css
	-rm  book_sans_serif.css
	-rm  book*.idv
	-rm  book*.lg
	-rm  book*.ncx
	-rm  book*.tmp
	-rm  book*.xref
	-rm  book*.html
	-rm  book*.fls
	-rm  book*.fdb_latexmk
	-rm  book*.bbl
	-rm  content.opf
	-rm  book*x.svg

.PHONY: clean
clean: clean_ebook_build clean_figures
	$(CLEAN) -output-directory=book_serif book.tex
	$(CLEAN) -output-directory=book_sans_serif book_sans_serif.tex
	-rm  book*/*.bbl
	-rm  book*/*.run.xml
	-rm -rf book*-epub/META-INF
	-rm -rf book*-epub/OEBPS
	-rm  book*-epub/mimetype

.PHONY: mrproper
mrproper: clean
	$(CLEAN) -C $(src_figures)
	$(CLEAN) -C -output-directory=book_serif book.tex
	$(CLEAN) -C -output-directory=book_sans_serif book_sans_serif.tex
	-rm figures/*.png
	rm -rf epub/
	rm -rf release/
	rm -rf book_serif/
	rm -rf book_sans_serif/
	rm -rf book-epub/
	rm -rf book_sans_serif-epub/

.PHONY: bake
bake: release_serif release_sans_serif

.PHONY: make_release_dir
make_release_dir:
	mkdir -p release

.PHONY: release_serif
release_serif: build_serif_pdf build_serif_ebook | make_release_dir
	cp book_serif/book.pdf release/TheBreadCode-The-Sourdough-Framework.pdf
	cp epub/book.mobi release/TheBreadCode-The-Sourdough-Framework.mobi
	cp epub/book.epub release/TheBreadCode-The-Sourdough-Framework.epub
	cp epub/book.azw3 release/TheBreadCode-The-Sourdough-Framework.azw3

.PHONY: release_sans_serif
release_sans_serif: build_sans_serif_pdf build_sans_serif_ebook | make_release_dir
	cp book_sans_serif/book_sans_serif.pdf  release/TheBreadCode-The-Sourdough-Framework-sans-serif.pdf
	cp epub/book_sans_serif.mobi release/TheBreadCode-The-Sourdough-Framework-sans-serif.mobi
	cp epub/book_sans_serif.epub release/TheBreadCode-The-Sourdough-Framework-sans-serif.epub
	cp epub/book_sans_serif.azw3 release/TheBreadCode-The-Sourdough-Framework-sans-serif.azw3

.PHONY: website
website:
	if command -v cowsay &> /dev/null; then\
		cowsay "baking the website";\
	fi
	rm -rf $(WEBSITE_DIR)
	make4ht -uf html5+tidy+common_domfilters -d $(WEBSITE_DIR) book.tex
	mv $(WEBSITE_DIR)/bookli1.html $(WEBSITE_DIR)/index.html

# Debug Stuff from now on
.PHONY: show_tools_version
show_tools_version:  # Show version of tools used on the build machine
	- latexmk --version
	@echo ""
	- pdflatex --version
	@echo ""
	- tex4ebook --version
	@echo ""
	- make4ht --version
	@echo ""
	- tidy -version
	@echo ""
	- kindlegen --version
	@echo ""
	- lacheck --version
	@echo ""
	- chktex --version
	@echo ""
	- make --version
	@echo ""

# You can find the value of variable X with the following command:
# make print-X
print-%: ; @echo $* = $($*)  # Print a makefile variable

.PHONY: printvars
printvars:  # Print all variables in the makefile
	@$(foreach V,$(sort $(.VARIABLES)), \
	$(if $(filter-out environ% default automatic, \
	$(origin $V)),$(info $V=$($V) ($(value $V)))))
