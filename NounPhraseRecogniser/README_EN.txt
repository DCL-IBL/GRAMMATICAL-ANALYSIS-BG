NounPhraseRecogniser
Recognition of noun phrases and adjective phrases in CoNLL-U text
(Bulgarian)

Department of Computational Linguistics
Institute for Bulgarian Language "Prof. Lyubomir Andreychin"
Bulgarian Academy of Sciences

Ivelina Stoyanova    iva@dcl.bas.bg
Svetlozara Leseva    zarka@dcl.bas.bg

Version 1.0, November 2026


1. PURPOSE
==========

NounPhraseRecogniser finds noun phrases headed by a noun (indefinite and
definite) and adjective phrases in a text annotated with UDPipe 2. For
each phrase it marks:

- its kind: indefinite noun phrase (NPI), definite noun phrase (NPD) or
  adjective phrase (AP);
- its head;
- the definiteness, gender and number of the phrase;
- the rule or rules by which it was recognised;
- the preposition, if the phrase is part of a prepositional phrase.

The result is added as a new column to the CoNLL-U file.

The rules and constraints follow the description of noun phrases in:
Osenova, P. (2009). Imennite frazi v balgarskiya ezik [Noun Phrases in
Bulgarian]. Sofia: ETO.


2. REQUIREMENTS
===============

- Java 8 or later (check: java -version).
- Input in CoNLL-U or CoNLL-U Plus format, annotated with UDPipe 2
  (model bulgarian-btb-ud-2.17-251125), with or without additional
  annotation:
  - if the input contains the column DCL:VERBFORM (VerbFormTagger), the
    words of verb forms are not included in noun phrases (e.g. the
    participle in "е написана");
  - if the input contains the column DCL:CLAUSE (ClauseSplitter),
    phrases do not cross clause boundaries.


3. USAGE
========

One file:
    java -jar NounPhraseRecogniser.jar -i text.conllu -o text.conllup

All .conllu and .conllup files in one folder
(the output files get the extension .conllup):
    java -jar NounPhraseRecogniser.jar -i conllu/ -o phrases/

Options:
    -i <file|folder>        input CoNLL-U (.conllu or .conllup)
    -o <file|folder>        output
    -resources <folder>     folder with resources that replace the
                            built-in ones (same file names)
    -gap <n>                maximum length of the (n,{...}) slots in the
                            rules (default: 2)
    -list                   prints the rules in order of application and
                            the active constraints, and exits

At the end the program prints the number of files, sentences and
phrases by kind.


4. RESOURCES
============

The resources are built into the jar file (folder resources/) and are
also supplied as separate files.

4.1 Rules (np_rules.tsv)

46 rules in the format: category, pattern, example, agreement: 15 for
indefinite noun phrases, 27 for definite noun phrases and 4 for
adjective phrases. They describe the combinations of the noun with
adjectives, possessive pronouns (full and short forms), cardinal and
ordinal numerals, demonstrative, interrogative, relative, indefinite,
negative and universal pronouns and quantity words. In addition, there
are 5 variants of main rules (e.g. майка ми, чаша вода, новата и
красива кола), which are not counted separately.

The rules are applied from longer to shorter (length = number of
obligatory components). Rules of the same length keep their order in
the file.

Pattern notation:

    Ni, Ad, PSi ...         tag from np_tags.tsv
    Ni!                     "!" marks the head
    (3,{Ai})                up to 3 optional words of the listed kinds
    и                       word: matches the lemma or the word form

A tag without a definiteness letter (N) matches all forms, and Nd
matches all definite forms.

4.2 Tags (np_tags.tsv)

Definition and description of the 36 tags (with examples) used in the
rules. They reflect the part of speech and the definiteness (indefinite
form, full and short article, definite form in the feminine, neuter and
plural). For each tag the corresponding BulTreeBank tags are given; the
program assigns the tags from the XPOS column on this basis.

4.3 Constraints (np_constraints.tsv)

7 constraints on the order of the components (2), the position of the
definite article (2) and of the short possessive form (2), and on
agreement in gender (and number). Every phrase found by the rules must
satisfy them. A constraint with "не" in the column "активно" is not
checked.


5. OUTPUT
=========

CoNLL-U Plus: the input columns are kept unchanged and one new column,
DCL:NP, is added, declared in the first line:

    # global.columns = ID FORM LEMMA UPOS XPOS FEATS HEAD DEPREL DEPS MISC DCL:NP

If the input is already CoNLL-U Plus (e.g. with the columns
DCL:VERBFORM and DCL:CLAUSE), DCL:NP is added after the existing
columns.

Values in the DCL:NP column:

    _       the word is not part of a phrase
    P<n>    the word is part of phrase n
    P<n>:<kind>[;prep=<preposition>];<features>;head=<ID>;rule=<id>[,<id>...]
            on the first word of phrase n

    kind       NPI, NPD or AP
    prep       the preposition, if the phrase is prepositional (the
               preposition is part of the phrase)
    features   Definite (Def if the phrase contains a definite word or
               a demonstrative pronoun), Gender and Number of the head
    head       ID of the head
    rule       the id of the rule (or rules)

The P numbers are unique within the output file.

Example:

    # text = Новата ми червена кола е в гаража.
    1   Новата   ...  P1:NPD;Definite=Def|Gender=Fem|Number=Sing;head=4;rule=NPD04
    2   ми       ...  P1
    3   червена  ...  P1
    4   кола     ...  P1
    5   е        ...  _
    6   в        ...  P2:NPD;prep=в;Definite=Def|Gender=Masc|Number=Sing;head=7;rule=NPD01
    7   гаража   ...  P2
    8   .        ...  _
