vectorsources := $(shell find rawsrcfigs/ -maxdepth 1 -name '*.dia' -o -name '*.eps' -o -name '*.m' -o -name '*.pdf' -o -name '*.ps' -o -name '*.py' -o -name '*.svg' 2>/dev/null)
rastersources := $(shell find rawsrcfigs/ -maxdepth 1 -name '*.gif' -o -name '*.webp' -o -name '*.avif' 2>/dev/null)
readysources  := $(shell find rawsrcfigs/ -maxdepth 1 -name '*.jpg' -o -name '*.png' -o -name '*.jpeg' -o -name '*.pdf' -o -name '*mpg' -o -name '*.mpeg' 2>/dev/null)

vectorfigs := $(shell echo ' ' $(patsubst %_fm.eps,%.eps,$(vectorsources)) ' ' | sed -e 's> \(../common/\)*raw> >g' -e 's/\.[^. ]* /.pdf /g')
rasterfigs := $(shell echo ' ' $(rastersources) ' ' | sed -e 's> \(../common/\)*raw> >g' -e 's/\.[^. ]* /.jpg /g')
readyfigs  := $(shell echo ' ' $(readysources) ' ' | sed -e 's> \(../common/\)*raw> >g')
figsources := $(vectorsources) $(rastersources) $(readysources) $(texfigsources)
figures    := $(vectorfigs) $(rasterfigs) $(readyfigs)


texsource := $(filter-out %figspreamble.tex, $(wildcard *.tex))
texfinal := $(patsubst %.tex,../../figs/%.pdf,$(texsource))

dirname  := $(shell basename $(shell pwd))

PDFLATEX ?= lualatex

INKSCAPE   ?= inkscape
FRAGMASTER ?= fragmaster
MYPDFLATEX   = TEXINPUTS=$(abspath ../common):$$TEXINPUTS $(PDFLATEX) --halt-on-error --interaction=batchmode
MYFRAGMASTER = TEXINPUTS=$(abspath ../common):$$TEXINPUTS $(FRAGMASTER)

.PHONY: all cleanlatex cleanfigs

all: $(texfinal)

clean: cleanlatex cleanfigs

cleanfigs:
	@rm -rf srcfigs/

cleanlatex:
	@rm -f ./*.aux ./*.log ./*.bbl ./*.blg ./*.dvi ./*.nav ./*.out ./*.snm ./*.toc ./*.vrb ./*.lof ./*.lot ./*.gnuplot ./*.table

continuous: all
	while true; do \
	  inotifywait -e close_write -e delete_self -e move $(all) || break; \
          $(MAKE) all; \
        done

$(texfinal): $(figures)

../../figs/%.pdf: %.tex
	@echo "$@ <- $(shell echo $? | cut -d' ' -f1)"
	@mkdir -p $(dir $@)
	@$(MYPDFLATEX) $(shell echo $? | cut -d' ' -f1) && $(MYPDFLATEX) $(shell echo $? | cut -d' ' -f1) && mv $(shell echo $@ | cut -d'/' -f4) ../../figs/


srcfigs/%.pdf: rawsrcfigs/%.dia
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@dia -t eps-builtin -e $?_roytemp.eps $? && epstopdf $?_roytemp.eps -o=$@
	@rm -f $?_roytemp.eps

srcfigs/%.pdf: rawsrcfigs/%.eps
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@epstopdf $? -o=$@ || (rm $@; exit 1)

srcfigs/%.pdf: rawsrcfigs/%_fm rawfigs/%_fm.eps  # fragmaster(1) with optional control file
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@(test -f rawsrcfigs/fragmaster.dfm && (cd $(dir $@) && ln -sf -t . ../rawfigs/fragmaster.dfm) || true)
	@(cd $(dir $@) && ln -sf -t . $(addprefix ../,$+) && $(MYFRAGMASTER))

srcfigs/%.pdf: rawsrcfigs/%.m
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@cd $(dir $?) && $(OCTAVE) $(notdir $?)
	@(test -f $(dir $?)/$*.eps && (epstopdf $(dir $?)/$*.eps -o=$@; rm -f $(dir $?)/$*.eps) || true)
	@(test -f $(dir $?)/$*.pdf && mv $(dir $?)/$*.pdf $@ || true)

srcfigs/%.pdf: rawsrcfigs/%.py
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@cd $(dir $?) && $(PYTHON) $(notdir $?)
	@(test -f $(dir $?)/$*.eps && (epstopdf $(dir $?)/$*.eps -o=$@; rm -f $(dir $?)/$*.eps) || true)
	@(test -f $(dir $?)/$*.pdf && mv $(dir $?)/$*.pdf $@ || true)

srcfigs/%.pdf: rawsrcfigs/%.ps
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@ps2pdf $? $@

srcfigs/%.pdf: rawsrcfigs/%.svg
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@$(INKSCAPE) $? --export-area-drawing --export-filename=$@

srcfigs/%.png: rawsrcfigs/%.png
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@gm convert -geometry "1920x1100>" $? $@

srcfigs/%.jpg: rawsrcfigs/%.gif
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@gm convert -geometry "1920x1100>" $? $@

srcfigs/%.jpg: rawsrcfigs/%.webp
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@gm convert -geometry "1920x1100>" $? $@

srcfigs/%.jpg: rawsrcfigs/%.avif
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@gm convert -geometry "1920x1100>" $? $@

srcfigs/%.mpeg: rawsrcfigs/%.mpeg
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@reldir=`echo $(dir $@) | sed -e 's>[^/]*/*>../>g'`; ln -sf $${reldir}$? $@

srcfigs/%.mpg: rawsrcfigs/%.mpg
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@reldir=`echo $(dir $@) | sed -e 's>[^/]*/*>../>g'`; ln -sf $${reldir}$? $@

srcfigs/%.pdf: rawsrcfigs/%.pdf
	@echo "$@ <- $(shell echo $? | cut -d' ' -f 1)"
	@mkdir -p $(dir $@)
	@reldir=`echo $(dir $@) | sed -e 's>[^/]*/*>../>g'`; ln -sf $${reldir}$(shell echo $? | cut -d' ' -f 1) $@

srcfigs/%.jpg: rawsrcfigs/%.jpg
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@gm convert -geometry "1920x1100>" $? $@

srcfigs/%.jpeg: rawsrcfigs/%.jpeg
	@echo "$@ <- $?"
	@mkdir -p $(dir $@)
	@gm convert -geometry "1920x1100>" $? $@

