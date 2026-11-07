AnaphoraResolver
Recognition of missing subjects and pronominal anaphora in CoNLL-U text
(Bulgarian)

Department of Computational Linguistics
Institute for Bulgarian Language "Prof. Lyubomir Andreychin"
Bulgarian Academy of Sciences

Ivelina Stoyanova    iva@dcl.bas.bg
Svetlozara Leseva    zarka@dcl.bas.bg

Version 1.0, November 2026


1. PURPOSE
==========

AnaphoraResolver:

- recognises clauses with a missing subject and assigns the type of
  the unexpressed subject: Pro-drop, Impersonal, Обобщено-личен
  (generic personal), Неопределено-личен (indefinite personal);
- recognises anaphoric pronouns;
- for pronouns and for missing subjects of type Pro-drop, marks the
  explicit referent in the same sentence, in the same or in the
  previous paragraph;
- builds coreference chains of noun phrase heads, pronouns and
  predicates with a missing subject.

The result is added as two new columns to the CoNLL-U file.


2. REQUIREMENTS
===============

- Java 8 or later (check: java -version).
- Input in CoNLL-U or CoNLL-U Plus format, annotated with UDPipe 2
  (model bulgarian-btb-ud-2.17-251125), with or without additional
  annotation (DCL:VERBFORM, DCL:CLAUSE, DCL:NP). Clauses, predicates and
  subjects are taken from the UDPipe dependency analysis. If the input
  contains the column DCL:NP (NounPhraseRecogniser), the heads of the
  noun phrases in it are used as referents.
- Paragraphs are taken from the "# newpar" lines and documents from the
  "# newdoc" lines. Referents are searched only within the document.


3. USAGE
========

One file:
    java -jar AnaphoraResolver.jar -i text.conllu -o text.conllup

All .conllu and .conllup files in one folder
(the output files get the extension .conllup):
    java -jar AnaphoraResolver.jar -i conllu/ -o anaphora/

Options:
    -i <file|folder>        input CoNLL-U (.conllu or .conllup)
    -o <file|folder>        output
    -resources <folder>     folder with resources that replace the
                            built-in ones (same file names)
    -list                   prints the rules and exits

At the end the program prints the number of files and sentences, the
number of missing subjects by type and the number of anaphora with and
without a referent found.


4. RESOURCES
============

The resources are built into the jar file (folder resources/) and are
also supplied as separate files.

4.1 Missing subject rules (subject_rules.tsv)

10 rules applied to the predicate of every clause without a subject.
The first rule whose conditions are met assigns the type of the
unexpressed subject:

    Impersonal (S01–S05)           impersonal verbs, има/няма,
                                   impersonal modal verbs with a
                                   да-clause, predicatives with "съм",
                                   impersonal use with "се"
    Обобщено-личен (S06)           2nd person singular with no addressee
                                   in the paragraph
    Неопределено-личен (S07)       3rd person plural with no possible
                                   referent
    Pro-drop (S08–S10)             1st and 2nd person; да-clause
                                   dependent on a predicate with the
                                   same person and number; 3rd person

4.2 Referent rules (anaphora_rules.tsv)

21 rules for pronouns and for missing subjects of type Pro-drop. For
each anaphor the rules are checked in the order of the file; the first
one that finds a referent applies. Each rule gives the kind of anaphor,
the strategy (e.g. the subject of the same or of the preceding clause,
the nearest preceding mention, the noun modified by the relative
clause), the scope (clause, sentence, paragraph, previous paragraph)
and additional conditions. The referent must always agree with the
anaphor in person, number and (in the singular) gender.

4.3 Pronoun forms (pronouns.tsv)

102 pronoun forms with kind, person, gender and number: personal (full
and short forms, nominative, accusative and dative), reflexive,
possessive (full and short forms), reflexive possessive, demonstrative
(used on their own) and relative. Forms with more than one kind (e.g.
му – personal or possessive) are distinguished by XPOS.

4.4 Impersonal verbs (impersonal_verbs.tsv)

Lists of impersonal verbs (вали, съмва се), existential има/няма,
impersonal modal verbs (трябва, предстои), verbs used impersonally with
"се" (говори се, смята се) and predicatives (студено, ясно, възможно).


5. OUTPUT
=========

CoNLL-U Plus: the input columns are kept unchanged and two new columns
are added, declared in the first line:

    # global.columns = ID FORM LEMMA UPOS XPOS FEATS HEAD DEPREL DEPS MISC DCL:ANAPHORA DCL:COREF

If the input is already CoNLL-U Plus, the columns are added after the
existing ones.

DCL:ANAPHORA – missing subjects and anaphora:

    Z<n>:<type>;<features>;rule=<id>[,<id>];ref=<referent>
            on the predicate (head verb) of a clause with a missing
            subject; type: Pro-drop, Impersonal, Обобщено-личен,
            Неопределено-личен
    A<n>:<kind>;<features>;rule=<id>;ref=<referent>
            on an anaphoric pronoun; kind: Personal, Possessive,
            Reflexive, ReflexivePossessive, Demonstrative, Relative
    _       other words

    features   person, number and gender of the unexpressed subject or
               of the pronoun
    rule       the rule for the subject type (S...) and/or the rule by
               which the referent was found (R...)
    ref        the referent: Z<n> or A<n> if it is another missing
               subject or anaphor, or <sent_id>#<ID> – the head of a
               noun phrase or a predicate in the sentence with the given
               sent_id; missing if no referent was found

DCL:COREF – coreference chains:

    E<n>    the word is part of coreference chain n: a noun phrase head,
            a pronoun, a predicate with a missing subject, or a
            predicate referred to by "това"
    _       other words

The Z, A and E numbers are unique within the output file.

Example:

    # text = Видях я и ѝ подадох книгата, която донесох.
    1   Видях     ...  Z3:Pro-drop;Number=Sing|Person=1;rule=S08                         E2
    2   я         ...  A2:Personal;Gender=Fem|Number=Sing|Person=3;rule=R12;ref=A1       E1
    3   и         ...  _                                                                 _
    4   ѝ         ...  A3:Personal;Gender=Fem|Number=Sing|Person=3;rule=R13;ref=A1       E1
    5   подадох   ...  Z4:Pro-drop;Number=Sing|Person=1;rule=S08,R05;ref=Z3              E2
    6   книгата   ...  _                                                                 E3
    7   ,         ...  _                                                                 _
    8   която     ...  A4:Relative;Gender=Fem|Number=Sing|Person=3;rule=R01;ref=4#6      E3
    9   донесох   ...  Z5:Pro-drop;Number=Sing|Person=1;rule=S08,R05;ref=Z4              E2
    10  .         ...  _                                                                 _

(A1 is "Тя" in the previous sentence; chain E1 starts with "Мария".)
