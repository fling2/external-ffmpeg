ifeq ($(CONFIG_AVCODEC),yes)
DOCLIBS += libavcodec
endif

MANPAGES    = $(PROGS-yes:%=doc/%.1)    $(DOCLIBS:%=doc/%.3)
PODPAGES    = $(PROGS-yes:%=doc/%.pod)  $(DOCLIBS:%=doc/%.pod)
HTMLPAGES   = $(PROGS-yes:%=doc/%.html) $(DOCLIBS:%=doc/%.html)         \
              doc/developer.html                                        \
              doc/faq.html                                              \
              doc/fate.html                                             \
              doc/general.html                                          \
              doc/git-howto.html                                        \
              doc/nut.html                                              \
              doc/platform.html                                         \
              doc/syntax.html                                           \

TXTPAGES    = doc/fate.txt                                              \


DOCS-$(CONFIG_HTMLPAGES) += $(HTMLPAGES)
DOCS-$(CONFIG_PODPAGES)  += $(PODPAGES)
DOCS-$(CONFIG_MANPAGES)  += $(MANPAGES)
DOCS-$(CONFIG_TXTPAGES)  += $(TXTPAGES)
DOCS = $(DOCS-yes)

all-$(CONFIG_DOC): doc

doc: documentation

apidoc: doc/doxy/html
documentation: $(DOCS)

TEXIDEP = awk '/^@(verbatim)?include/ { printf "$@: $(@D)/%s\n", $$2 }' <$< >$(@:%=%.d)

doc/%.txt: TAG = TXT
doc/%.txt: doc/%.texi
	$(Q)$(TEXIDEP)
	$(M)makeinfo --force --no-headers -o $@ $< 2>/dev/null

GENTEXI  = format codec
GENTEXI := $(GENTEXI:%=doc/avoptions_%.texi)

$(GENTEXI): TAG = GENTEXI
$(GENTEXI): doc/avoptions_%.texi: doc/print_options$(HOSTEXESUF)
	$(M)doc/print_options $* > $@

doc/%.html: TAG = HTML
doc/%.html: doc/%.texi $(SRC_PATH)/doc/t2h.init $(GENTEXI)
	$(Q)$(TEXIDEP)
	$(M)texi2html -I doc -monolithic --init-file $(SRC_PATH)/doc/t2h.init --output $@ $<

doc/%.pod: TAG = POD
doc/%.pod: doc/%.texi $(GENTEXI)
	$(Q)$(TEXIDEP)
	$(M)perl $(SRC_PATH)/doc/texi2pod.pl -Idoc $< $@

doc/%.1 doc/%.3: TAG = MAN
doc/%.1 doc/%.3: doc/%.pod $(GENTEXI)
	$(M)pod2man --section=1 --center=" " --release=" " $< > $@

$(DOCS) doc/doxy/html: | doc/

doc/doxy/html: $(SRC_PATH)/doc/Doxyfile $(INSTHEADERS)
	$(M)$(SRC_PATH)/doc/doxy-wrapper.sh $(SRC_PATH) $^

install-man:

ifdef CONFIG_MANPAGES
install-progs-$(CONFIG_DOC): install-man

install-man: $(MANPAGES)
	$(Q)mkdir -p "$(MANDIR)/man1"
	$(INSTALL) -m 644 $(MANPAGES) "$(MANDIR)/man1"
endif

uninstall: uninstall-man

uninstall-man:
	$(RM) $(addprefix "$(MANDIR)/man1/",$(ALLMANPAGES))

clean::
	$(RM) $(TXTPAGES) doc/*.html doc/*.pod doc/*.1 $(CLEANSUFFIXES:%=doc/%) doc/avoptions_*.texi
	$(RM) -r doc/doxy/html

-include $(wildcard $(DOCS:%=%.d))

.PHONY: apidoc doc documentation
