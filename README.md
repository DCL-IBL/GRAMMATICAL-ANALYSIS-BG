# Grammatical Annotation Tools for Bulgarian

Rule-based tools for the annotation of Bulgarian (and partly English) texts on top of [UDPipe 2](https://lindat.mff.cuni.cz/services/udpipe/) output. Each tool reads CoNLL-U / CoNLL-U Plus and adds its own column, so the tools can be used separately or chained.

**[English](#english) | [Български](#български)**

---

## English

### Tools

| Tool | Version | Languages | Adds column(s) | What it does |
|---|---|---|---|---|
| **VerbFormTagger** | 1.0 | bg, en | `VERBFORM` | Recognises simple and analytic (incl. discontinuous) verb forms and labels grammatical features of the full form |
| **ClauseSplitter** | 2.0 | bg, en | `CLAUSE` | Splits complex sentences into clauses |
| **NounPhraseRecogniser** | 1.0 | bg | `NP` | Recognises indefinite and definite noun phrases and adjective phrases |
| **AnaphoraResolver** | 1.0 | bg | `ANAPHORA`, `COREF` | Recognises missing subjects (Pro-drop, Impersonal, generic and indefinite personal), resolves pronominal anaphora and builds coreference chains |

ClauseSplitter 2.0 replaces [ClauseSplitter 1.0 (2017)](https://dcl.bas.bg/tools_list/clausesplitter-tool-for-identification-of-clauses-within-the-sentence/).

### Pipeline

```
text ──► UDPipe 2 ──► VerbFormTagger ──► ClauseSplitter ──► NounPhraseRecogniser ──► AnaphoraResolver
          .conllu       + DCL:VERBFORM    + DCL:CLAUSE       + DCL:NP                 + DCL:ANAPHORA, DCL:COREF
```

Every tool accepts plain UDPipe output as well as the output of the previous tools. Existing columns are kept, and new columns are declared in `# global.columns`. Where available, earlier annotation is used:
- ClauseSplitter takes the verb forms from `DCL:VERBFORM`.
- NounPhraseRecogniser keeps phrases within clauses and leaves out words that belong to verb forms.
- AnaphoraResolver takes its referent candidates from `DCL:NP`.

### Requirements

- Java 8 or later
- Input annotated with UDPipe 2: `bulgarian-btb-ud-2.17-251125` / `english-ewt-ud-2.17-251125`

### Quick start

```bash
java -jar VerbFormTagger/VerbFormTagger.jar             -i text.conllu      -o text_vf.conllup
java -jar ClauseSplitter/ClauseSplitter.jar             -i text_vf.conllup  -o text_cl.conllup
java -jar NounPhraseRecogniser/NounPhraseRecogniser.jar -i text_cl.conllup  -o text_np.conllup
java -jar AnaphoraResolver/AnaphoraResolver.jar         -i text_np.conllup  -o text_all.conllup
```

All tools accept a file or a folder (`-i conllu/ -o out/`), and `-list` prints the rules. VerbFormTagger and ClauseSplitter take `-lang en` for English. With `-resources <folder>`, NounPhraseRecogniser and AnaphoraResolver (and ClauseSplitter) use edited copies of their resources instead of the built-in ones.

### Repository contents

Each tool folder has the same layout:

```
<Tool>/
  <Tool>.jar            executable jar (resources built in)
  README_BG.txt         documentation in Bulgarian
  README_EN.txt         documentation in English
  resources/ or rules/  resource files (also built into the jar)
  src/                  Java source
  examples/             example input and output
```

### Resources

All resources are tab-separated UTF-8 files with a header line.

**VerbFormTagger** (`rules/`)

| File | Content |
|---|---|
| `verb_form_rules_bg_full.tsv` | 823 rules for all Bulgarian verb forms |
| `verb_form_rules_bg_basic.tsv` | 245 rules for the basic forms: indicative tenses, imperative, conditional, passive, negation |
| `verb_form_rules_en_full.tsv` | 136 rules for English verb forms |
| `rules_cleaning_report.txt`, `rules_cleaning_log.txt` | how the rule sets were checked and cleaned |

**ClauseSplitter** (`resources/`)

| File | Content |
|---|---|
| `clause_rules.tsv` | 8 splitting rules: 7 between adjacent verb forms and 1 before the first verb form |
| `conj_bg.tsv`, `conj_en.tsv` | words and expressions introducing a clause (316 Bulgarian, 226 English), marked as strong or weak |
| `special_verbs_bg.tsv`, `special_verbs_en.tsv` | 24 modal, phase and auxiliary verbs after which a *да*-construction is not a separate clause |
| `strong_punct_exceptions.tsv` | cases where strong punctuation is not a clause boundary |
| `verb_form_rules_bg_full.tsv`, `verb_form_rules_en_full.tsv` | verb form rules, used when the input has no `DCL:VERBFORM` |

**NounPhraseRecogniser** (`resources/`)

| File | Content |
|---|---|
| `np_rules.tsv` | 46 rules (15 indefinite NP, 27 definite NP, 4 AP) and 5 variants |
| `np_tags.tsv` | 36 tags for part of speech and definiteness, mapped to BulTreeBank XPOS |
| `np_constraints.tsv` | 7 constraints on word order, the position of the article and of the short possessive, and agreement |

**AnaphoraResolver** (`resources/`)

| File | Content |
|---|---|
| `subject_rules.tsv` | 10 rules for clauses with a missing subject and its type |
| `anaphora_rules.tsv` | 21 rules for finding the referent of pronouns and of Pro-drop subjects |
| `pronouns.tsv` | 102 pronoun forms with kind, person, gender and number |
| `impersonal_verbs.tsv` | impersonal verbs, existential *има/няма*, impersonal modals, impersonal *се*-verbs, predicatives |

### Output columns

| Column | Example (first word of a unit) | Other words |
|---|---|---|
| `DCL:VERBFORM` | `F11:Future_Perfect_in_the_Past+Neg;<features>;rule=BGF…` | `F11` |
| `DCL:CLAUSE` | `C4:SUB;head=C3;conj=S016:че` | `C4` |
| `DCL:NP` | `P1:NPD;Definite=Def\|Gender=Fem\|Number=Sing;head=4;rule=NPD04` | `P1` |
| `DCL:ANAPHORA` | `Z3:Pro-drop;…;rule=S08,R05;ref=Z2` (on the verb), `A2:Personal;…;rule=R12;ref=A1` (on the pronoun) | `_` |
| `DCL:COREF` | `E1` on every member of coreference chain 1 | `_` |

IDs are unique within each output file. See each tool's README for details.

 
Department of Computational Linguistics, Institute for Bulgarian Language, Bulgarian Academy of Sciences

 

The UDPipe 2 models used to produce the input are distributed under CC BY-NC-SA 4.0.

---

## Български

### Инструменти

| Инструмент | Версия | Езици | Добавя колона | Предназначение |
|---|---|---|---|---|
| **VerbFormTagger** | 1.0 | bg, en | `VERBFORM` | Разпознава простите и аналитичните (вкл. прекъснатите) глаголни форми и определя граматическите характеристики на цялата форма |
| **ClauseSplitter** | 2.0 | bg, en | `CLAUSE` | Разделя сложното изречение на прости изречения |
| **NounPhraseRecogniser** | 1.0 | bg | `NP` | Разпознава нечленувани и членувани именни фрази и прилагателни фрази |
| **AnaphoraResolver** | 1.0 | bg | `ANAPHORA`, `COREF` | Разпознава изпуснатия подлог (Pro-drop, Impersonal, обобщено-личен, неопределено-личен), намира референта на местоименните анафори и изгражда кореферентни вериги |

ClauseSplitter 2.0 заменя [ClauseSplitter 1.0 (2017)](https://dcl.bas.bg/tools_list/clausesplitter-tool-for-identification-of-clauses-within-the-sentence/).

### Последователност на обработката

```
текст ──► UDPipe 2 ──► VerbFormTagger ──► ClauseSplitter ──► NounPhraseRecogniser ──► AnaphoraResolver
           .conllu       + DCL:VERBFORM    + DCL:CLAUSE       + DCL:NP                 + DCL:ANAPHORA, DCL:COREF
```

Всеки инструмент приема както резултата от UDPipe, така и резултата от предходните инструменти. Наличните колони се запазват, а новите се обявяват в `# global.columns`. Когато има по-ранна анотация, тя се използва:
- ClauseSplitter взема глаголните форми от `DCL:VERBFORM`.
- NounPhraseRecogniser не допуска фразите да преминават границите на простите изречения и не включва в тях думи от глаголни форми.
- AnaphoraResolver взема кандидатите за референт от `DCL:NP`.

### Изисквания

- Java 8 или по-нова версия
- Вход, анотиран с UDPipe 2: `bulgarian-btb-ud-2.17-251125` / `english-ewt-ud-2.17-251125`

### Бърз старт

```bash
java -jar VerbFormTagger/VerbFormTagger.jar             -i text.conllu      -o text_vf.conllup
java -jar ClauseSplitter/ClauseSplitter.jar             -i text_vf.conllup  -o text_cl.conllup
java -jar NounPhraseRecogniser/NounPhraseRecogniser.jar -i text_cl.conllup  -o text_np.conllup
java -jar AnaphoraResolver/AnaphoraResolver.jar         -i text_np.conllup  -o text_all.conllup
```

Всички инструменти приемат файл или папка (`-i conllu/ -o out/`), а `-list` извежда правилата. VerbFormTagger и ClauseSplitter приемат `-lang en` за английски. С `-resources <папка>` NounPhraseRecogniser и AnaphoraResolver (както и ClauseSplitter) използват редактирани копия на ресурсите вместо вградените.

### Съдържание на хранилището

Папката на всеки инструмент има една и съща структура:

```
<Инструмент>/
  <Инструмент>.jar      изпълним jar файл (с вградени ресурси)
  README_BG.txt         документация на български
  README_EN.txt         документация на английски
  resources/ или rules/ ресурси (вградени и в jar файла)
  src/                  изходен код на Java
  examples/             примерен вход и резултат
```

### Ресурси

Всички ресурси са файлове с табулации (UTF-8) със заглавен ред.

**VerbFormTagger** (`rules/`)

| Файл | Съдържание |
|---|---|
| `verb_form_rules_bg_full.tsv` | 423 правила за всички глаголни форми в българския език |
| `verb_form_rules_bg_basic.tsv` | 110 правила за основните форми: времена на изявителното наклонение, повелително и условно наклонение, страдателен залог, отрицание |
| `verb_form_rules_en_full.tsv` | 136 правила за глаголните форми в английския език |
| `rules_cleaning_report.txt`, `rules_cleaning_log.txt` | как са проверени и изчистени правилата |

**ClauseSplitter** (`resources/`)

| Файл | Съдържание |
|---|---|
| `clause_rules.tsv` | 8 правила за разделяне: 7 между съседни глаголни форми и 1 преди първата глаголна форма |
| `conj_bg.tsv`, `conj_en.tsv` | думи и изрази, въвеждащи просто изречение (316 на български, 226 на английски), с разграничение „силни“ / „слаби“ |
| `special_verbs_bg.tsv`, `special_verbs_en.tsv` | 24 модални, фазови и спомагателни глагола, след които да-конструкцията не е самостоятелно просто изречение |
| `strong_punct_exceptions.tsv` | случаи, в които силният препинателен знак не е граница |
| `verb_form_rules_bg_full.tsv`, `verb_form_rules_en_full.tsv` | правила за глаголни форми, ако във входа няма колона `DCL:VERBFORM` |

**NounPhraseRecogniser** (`resources/`)

| Файл | Съдържание |
|---|---|
| `np_rules.tsv` | 46 правила (15 за нечленувани, 27 за членувани именни фрази, 4 за прилагателни фрази) и 5 варианта |
| `np_tags.tsv` | 36 тага за част на речта и определеност със съответствията им в XPOS на BulTreeBank |
| `np_constraints.tsv` | 7 ограничения за реда на думите, мястото на члена и на кратката притежателна форма и съгласуването |

**AnaphoraResolver** (`resources/`)

| Файл | Съдържание |
|---|---|
| `subject_rules.tsv` | 10 правила за простите изречения с изпуснат подлог и за типа му |
| `anaphora_rules.tsv` | 21 правила за намиране на референта на местоименията и на изпуснатия подлог (Pro-drop) |
| `pronouns.tsv` | 102 местоименни форми с вид, лице, род и число |
| `impersonal_verbs.tsv` | безлични глаголи, битийните *има/няма*, безлични модални глаголи, глаголи в безлична употреба със „се“, предикативи |

### Колони в резултата

| Колона | Пример (първа дума на единицата) | Останалите думи |
|---|---|---|
| `DCL:VERBFORM` | `F11:Future_Perfect_in_the_Past+Neg;<признаци>;rule=BGF…` | `F11` |
| `DCL:CLAUSE` | `C4:SUB;head=C3;conj=S016:че` | `C4` |
| `DCL:NP` | `P1:NPD;Definite=Def\|Gender=Fem\|Number=Sing;head=4;rule=NPD04` | `P1` |
| `DCL:ANAPHORA` | `Z3:Pro-drop;…;rule=S08,R05;ref=Z2` (на глагола), `A2:Personal;…;rule=R12;ref=A1` (на местоимението) | `_` |
| `DCL:COREF` | `E1` на всеки член на кореферентна верига 1 | `_` |

Идентификаторите са уникални в рамките на всеки изходен файл. Подробности има в README файловете на отделните инструменти.
 

Секция по компютърна лингвистика, Институт за български език „Проф. Любомир Андрейчин“, Българска академия на науките
 

Моделите на UDPipe 2, с които се получава входът, се разпространяват с лиценз CC BY-NC-SA 4.0.
