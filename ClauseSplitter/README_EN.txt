ClauseSplitter
Splitting complex sentences into clauses in CoNLL-U text
(Bulgarian and English)

Department of Computational Linguistics
Institute for Bulgarian Language "Prof. Lyubomir Andreychin"
Bulgarian Academy of Sciences

Ivelina Stoyanova    iva@dcl.bas.bg
Svetlozara Leseva    zarka@dcl.bas.bg

Version 2.0, November 2026

Previous version:
ClauseSplitter – tool for identification of clauses within the sentence,
version 1.0, 2017.
https://dcl.bas.bg/tools_list/clausesplitter-tool-for-identification-of-clauses-within-the-sentence/


1. PURPOSE
==========

ClauseSplitter splits each sentence in a text annotated with UDPipe 2
into clauses. For each clause it marks:

- its kind: main (MAIN), coordinated (COORD) or subordinate (SUB);
- the clause it is subordinated or coordinated to;
- the conjunction or connective word that introduces it.

The result is added as a new column to the CoNLL-U file. ClauseSplitter
does not mark verb forms.


2. REQUIREMENTS
===============

- Java 8 or later (check: java -version).
- Input in CoNLL-U or CoNLL-U Plus format, annotated with UDPipe 2:
    Bulgarian: bulgarian-btb-ud-2.17-251125
    English:   english-ewt-ud-2.17-251125
  with or without marked analytic verb forms:
  - if the input contains the column DCL:VERBFORM (output of
    VerbFormTagger), the forms marked in it are used;
  - if not, the verb forms are found internally with the full lists of
    forms of VerbFormTagger, for the purposes of splitting only. They are
    not written to the output.


3. USAGE
========

One file:
    java -jar ClauseSplitter.jar -i text.conllu -o text.conllup

All .conllu files in one folder
(the output files get the extension .conllup):
    java -jar ClauseSplitter.jar -i conllu/ -o clauses/

English text:
    java -jar ClauseSplitter.jar -lang en -i text_en.conllu -o text_en.conllup

Options:
    -i <file|folder>        input CoNLL-U (.conllu or .conllup)
    -o <file|folder>        output
    -lang bg|en             language of the input (default: bg)
    -resources <folder>     folder with resources that replace the
                            built-in ones (same file names)
    -gap <n>                maximum length of the gaps in the verb form
                            rules (default: 3)
    -list                   prints the clause splitting rules and exits

At the end the program prints the number of files, sentences, verb forms
and clauses.


4. RESOURCES
============

The resources are built into the jar file (folder resources/) and are
also supplied as separate files.

4.1 Words and expressions introducing a clause
(conj_bg.tsv, conj_en.tsv)

A dictionary of words and expressions that mark the beginning of a
clause (e.g. because, when, to this end, besides, moreover, therefore)
in Bulgarian and English, with parallel translation equivalents: 316
words and expressions for Bulgarian and 226 for English. They are
classified as "strong" (always introduce a clause) and "weak" (may not
introduce a clause, e.g. the conjunction "and", which can also join
coordinated parts of a clause).

4.2 Verbs with special meaning and use
(special_verbs_bg.tsv, special_verbs_en.tsv)

Verbs after which the да-construction forms a compound verbal predicate
rather than a separate clause: 24 verbs for Bulgarian (8 modal – мога,
трябва, следва, смея, etc.; 12 phase verbs – 6 for the beginning, 2 for
the continuation and 4 for the end of the action; 4 auxiliary) with
their English equivalents.

4.3 Splitting rules (clause_rules.tsv)

8 rules applied in sequence: 7 in the interval between every two
adjacent verb forms and 1 before the first verb form. They use 6 types
of conditions for a position in the sentence to be the beginning of a
clause: verbs with special function, strong punctuation, strong
conjunctions, closing comma, weak conjunctions, weak punctuation and
a boundary before the next verb form. The order, conditions and actions
of the rules are described at the beginning of the file.

4.4 Other resources

    strong_punct_exceptions.tsv     cases in which a strong punctuation
                                    mark is not a boundary
                                    (abbreviations, ellipsis, etc.)
    verb_form_rules_bg_full.tsv,
    verb_form_rules_en_full.tsv     the full lists of verb forms of
                                    VerbFormTagger (used when the input
                                    has no DCL:VERBFORM column)


5. OUTPUT
=========

CoNLL-U Plus: the input columns are kept unchanged and one new column,
DCL:CLAUSE, is added, declared in the first line:

    # global.columns = ID FORM LEMMA UPOS XPOS FEATS HEAD DEPREL DEPS MISC DCL:CLAUSE

If the input is already CoNLL-U Plus (e.g. with the column
DCL:VERBFORM), DCL:CLAUSE is added after the existing columns.

Values in the DCL:CLAUSE column:

    C<n>                    the word is part of clause n
    C<n>:<kind>[;head=C<m>][;coord=C<m>][;conj=<id>:<conjunction>]
                            on the first word of clause n

    kind    MAIN (main), COORD (coordinated) or SUB (subordinate)
    head    the clause it is subordinated to
    coord   the clause it is coordinated with
    conj    id of the conjunction in the resource and its words in the text

The C numbers are unique within the output file.

Example (input with the DCL:VERBFORM column):

    # text = Той каза, че ще дойде, и си тръгна.
    1   Той     …  _                                     C3:MAIN
    2   каза    …  F3:Aorist;…;rule=BGF418               C3
    3   ,       …  _                                     C3
    4   че      …  _                                     C4:SUB;head=C3;conj=S016:че
    5   ще      …  F4:Future;…;rule=BGF391               C4
    6   дойде   …  F4                                    C4
    7   ,       …  _                                     C4
    8   и       …  _                                     C5:COORD;coord=C3;conj=S154:и
    9   си      …  F5:Aorist;…;rule=BGF418               C5
    10  тръгна  …  F5                                    C5
    11  .       …  _                                     C5
