VerbFormTagger
Recognition of analytic verb forms in CoNLL-U text
(Bulgarian and English)

Department of Computational Linguistics
Institute for Bulgarian Language "Prof. Lyubomir Andreychin"
Bulgarian Academy of Sciences

Ivelina Stoyanova    iva@dcl.bas.bg
Svetlozara Leseva    zarka@dcl.bas.bg

Version 1.0, November 2026


1. PURPOSE
==========

VerbFormTagger finds all verb forms in a text annotated with UDPipe 2.
It recognises both simple forms (дам, писах) and analytic forms
(щеше да е заминал, has not been working), including discontinuous ones
(нямаше по това време да се е върнал). For each form it marks:

- the grammatical form (tense, mood, voice, negation, evidentiality);
- the full morphological tag of the form as a whole;
- the rule or rules by which it was recognised.

The result is added as a new column to the CoNLL-U file.


2. REQUIREMENTS
===============

- Java 8 or later (check: java -version).
- Input annotated with UDPipe 2:
    Bulgarian: bulgarian-btb-ud-2.17-251125 (BulTreeBank tagset in XPOS)
    English:   english-ewt-ud-2.17-251125 (Penn Treebank tagset in XPOS)


3. USAGE
========

Main file:
    java -jar VerbFormTagger.jar -i text.conllu -o text.conllup

All .conllu files should be in one common folder
(the output files get the extension .conllup):
    java -jar VerbFormTagger.jar -i conllu/ -o verbforms/

To annotate English text:
    java -jar VerbFormTagger.jar -lang en -i text_en.conllu -o text_en.conllup

Options:
    -i <file|folder>   input CoNLL-U (.conllu or .conllup)
    -o <file|folder>   output
    -lang bg|en        language of the input (default: bg)
    -set basic|full    built-in rule set for Bulgarian
                       (default: basic)
    -rules <file>      external rules file; takes precedence over -set
    -gap <n>           maximum length of the open gaps
                       (n,{...}) in the rules (default: 3)
    -list              prints the rules in order of application and exits

At the end the program prints the number of files, sentences and verb
forms, as well as how many of the forms are analytic and how many are
ambiguous.


4. RESOURCES
============

4.1 Rule files

The rules are built into the jar file (folder rules/) and are also
supplied as separate files:

    verb_form_rules_bg_basic.tsv   245 rules, Bulgarian, main forms (110 generalised patterns)
    verb_form_rules_bg_full.tsv    823 rules, Bulgarian, all forms (423 generalised patterns)
    verb_form_rules_en_full.tsv    136 rules, English


The rule files use tab as a separator and have a header line:

    id        rule identifier (BGB001, BGF001, ENF001 ...)
    label     the form. Alternatives are separated with "|", and tenses
              with the same additional features with "-"
              (Present-Perfect+Pass = Present+Pass or Perfect+Pass)
    pattern   the components of the form, separated by spaces
    example   example (optional)
    frequency frequency (optional)

The rules are ordered from longer to shorter (length = number of
obligatory components) and are applied in this order. Rules of the same
length keep their order as given in the file.

4.2 Pattern notation

    да, не, ще, нека, not, to   word: matches the lemma or the word form
    съм:Vr3s                    lemma + tag
    Vxo                         tag only
    (5,{ли,се,си,P,да})         up to 5 words, each of which is one of
                                those listed or is of category P
                                (pronoun)
    (n,{-V,-U,-C})              open gap: up to n words (option -gap)
                                that are not verbs (V), punctuation (U)
                                or conjunctions (C)

A tag consists of a category (capital letter) and grammatical letters.
A word matches if its tag starts with the category and contains all the
letters. For example, Vr matches Vr1s, Vr3s, Vr3p, etc.


5. OUTPUT
=========

CoNLL-U Plus: the input columns are kept unchanged and one new column is
added, declared in the first line:

    # global.columns = ID FORM LEMMA UPOS XPOS FEATS HEAD DEPREL DEPS MISC DCL:VERBFORM

If the input is already CoNLL-U Plus, the column is added after the
existing ones.

Values in the DCL:VERBFORM column:

    _               the word is not part of a verb form
    F<n>            the word is a component of form n
    F<n>:<label>;<features>;rule=<id>[,<id>...]
                    on the first (in word order) component of form n

The F numbers are unique within the output file.


Example:

    # text = Той нямаше по това време да се е върнал.
    1   Той     …  _
    2   нямаше  …  F11:Future_Perfect_in_the_Past+Neg;Aspect=Perf|Gender=Masc|Mood=Ind|Number=Sing|Person=3|Polarity=Neg|Tense=FuturePerfectInThePast|VerbForm=Fin|Voice=Act;rule=BGB019
    3   по      …  _
    4   това    …  _
    5   време   …  _
    6   да      …  F11
    7   се      …  F11
    8   е       …  F11
    9   върнал  …  F11
    10  .       …  _

Ambiguous form (full rule set):

    е         F8:Perfect|Aorist+Concl;Aspect=Perf|…|Tense=Perfect|VerbForm=Fin|Voice=Act/Aspect=Perf|Evident=Concl|…|Tense=Aorist|VerbForm=Fin|Voice=Act;rule=BGF382
    изпуснал  F8
