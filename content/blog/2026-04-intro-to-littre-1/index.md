+++
title = "Deep-Littré: a walkthrough and technical reference"
date = 2026-04-12
description = "A quick introduction to the Deep-Littré: a deeply structured, computationally enriched edition of Émile Littré's French dictionary, available as TEI Lex-0 XML and SQLite."

[taxonomies]
tags = ["tools"]
+++

## Introduction
The four quarto volumes and supplement of Émile Littré's _Dictionnaire de la langue française_, published between 1862 and 1877, come to over 11 million words in total. That's longer than all the hundred novels of the [ELTeC-Fra](https://github.com/myersm0/isosceles?tab=readme-ov-file#structure) corpus combined.

Thankfully we have digital versions. François Gannaz's [XMLittré](https://www.littre.org/) project provides both a plain-text and an XML digitization of the full dictionary. In the XML edition, the internal structure of entries is represented by roughly 87,000 sub-entry blocks encoded as "\<indent\>" elements.

But the variety and structure of Littré's actual text are not fully captured by this flat, undifferentiated _indent_ structure. Specifically, a typical entry in Littré contains a hierarchy of several kinds of information: numbered senses (the core definitions), figurative extensions, domain labels, register labels, locutions, literary citations, etymologies, and historical notes.

While XMLittré preserves the text, it does not distinguish the different structural roles those elements play within an entry.

This is not a criticism of Gannaz’s work. Converting Littré into XML was a major undertaking that reportedly required hundreds or even thousands of hours. My own work, still in progress, has been a similarly meticulous task that builds directly on that effort.

My project introduced here, **[Deep-Littré](https://github.com/myersm0/deep-littre)**, is an ongoing attempt to recover the internal structure of Littré’s entries and represent it explicitly. The goal is to make the dictionary easier to analyze computationally and more navigable for readers.

## An small example entry: TRONQUER

### A walkthrough of the TEI format
The full TRONQUER entry is available [here](tronquer.txt) if you want to see it all at once. I'll walk through it piece by piece below.

#### Headword and part of speech
Every entry begins with the lemma, its pronunciation, and its grammatical category:
```xml
<entry xml:id="tronquer">
  <form type="lemma">
    <orth>TRONQUER</orth>
    <pron>tron-ké</pron>
  </form>
  <gramGrp><gram type="pos">v. a.</gram></gramGrp>
```

The `xml:id` is an ASCII-normalized key derived from the headword, useful for cross-references and as a stable identifier in the database. "v. a." is Littré's abbreviation for _verbe actif_ (a transitive verb).

#### Numbered senses
The definitions, ordered from concrete to abstract:
```xml
<sense xml:id="tronquer_s1" n="1">
  <def>Retrancher, couper.</def>
</sense>
<sense xml:id="tronquer_s2" n="2">
  <def>Scier sur le tour.</def>
</sense>
<sense xml:id="tronquer_s3" n="3">
  <usg type="sem">fig.</usg>
  <def>
    En parlant des ouvrages d'esprit et en mauvaise part, 
    y retrancher quelque chose d'essentiel.
  </def>
  [… citations omitted for brevity …]
</sense>
```

Each sense carries an xml:id and a number. The xml:id follows a hierarchical scheme: tronquer_s1 for the first sense, tronquer_s1.1 for its first sub-entry, and so on. (The sub-entries are not shown here.)

#### Literary citations
Most senses include one or more citations drawn from literature:

```xml
<cit type="example">
  <quote>
    Il ôte de chez lui les branches les plus belles,
    Il tronque son verger contre toute raison
  </quote>
  <bibl>
    <author>LA FONT.</author>
    <biblScope>Fabl. XII, 20</biblScope>
  </bibl>
</cit>
```

The `<author>` field preserves Littré's abbreviated forms: here, "LA FONT." for La Fontaine. 

Littré frequently uses `ID.` (idem) to mean "same author as above." Deep-Littré resolves these back to the actual author name.

#### Locutions
Fixed expressions with their own definitions, nested under the sense they belong to:

```xml
<re type="locution" xml:id="tronquer_s1.1">
  <form><orth>En parlant des statues</orth></form>
  <def>
    En parlant des statues, mutiler en partie.
    Les barbares ont tronqué la plupart des statues de Rome.
  </def>
</re>
```

A `<re>` tag (related entry) of `type="locution"` contains a _form_ (the canonical expression) and a _def_. This is a simple example; larger entries like MAIN or FAIRE contain dozens of locutions.

#### Usage labels
Sense 3 carries a label marking it as figurative:

```xml
<sense xml:id="tronquer_s3" n="3">
  <usg type="sem">fig.</usg>
  <def>
    En parlant des ouvrages d'esprit et en mauvaise part,
    y retrancher quelque chose d'essentiel.
  </def>
```

The `<usg>` element encodes usage information with a typed vocabulary: 
- `type="sem"` for semantic labels like "Fig." or "Figurément"
- `type="domain"` for field markers like "Terme de marine" or  "Terme de musique"
- `type="register"` for stylistic labels like "Familièrement" or "Populairement" 
- `type="gram"` for grammatical notes like "Absolument" or "Substantivement"

A couple of those terms warrant explanation:

Absolument: "used absolutely," meaning the verb or noun is used without its usual complement. If tronquer normally takes a direct object ("tronquer un texte"), then "absolument" signals the cases where it stands alone: "il ne fait que tronquer."

Substantivement: "used as a noun," signaling that a word whose primary part of speech is something else (e.g. adjective) is being used nominally.

Both are what Littré uses to mark a grammatical shift in how the headword behaves. They indicate a new syntactic context that brings its own set of senses. That's why Deep-Littré treats them as transition groups: they scope over everything that follows, establishing a grammatical frame for the sub-entries beneath them.

#### Historical notes and etymology
Most entries close with two sections. The _historique_ provides dated attestations, usually organized by century (XVe siècle, i.e. the 15th century in this case):

```xml
<note type="historique">
  <p>XVe s.</p>
  <cit type="example">
    <quote>
      Icellui Perrenet se print à copper et troncer
      lesdiz ormes
    </quote>
    <bibl>
      <author>DU CANGE</author>
      <biblScope>troncire.</biblScope>
    </bibl>
  </cit>
```

And _étymologie_:
```xml
<etym>
  <p>
    Provenç. et espagn. troncar ; ital. troncare ;
    du latin truncare (voy. TRONC).
  </p>
</etym>
```

The `voy.` ("voyez" — see) is a cross-reference to another entry. In the TEI these are encoded as `<xr><ref target="#tronc">`.


### The same data in SQL
The same entry in the SQL representation:
 
```sql
SELECT headword, pos, pronunciation
FROM entries
WHERE headword = 'TRONQUER';
```
```
TRONQUER | v. a. | tron-ké
```
 
Senses and their sub-entries live in a single `senses` table, linked by `parent_sense_id`:
 
```sql
SELECT indent_id, role, content_plain
FROM senses s
JOIN entries e ON e.entry_id = s.entry_id
WHERE e.headword = 'TRONQUER';
```
```
tronquer.1   | Definition  | Retrancher, couper.
tronquer.1.1 | Locution    | En parlant des statues, mutiler en partie. …
tronquer.2   | Definition  | Scier sur le tour.
tronquer.3   | Figurative  | En parlant des ouvrages d'esprit et en mauvaise …
```
 
The `indent_id` encodes the hierarchical position: `tronquer.1.1` is the first sub-entry under sense 1. The `role` column is the structural classification that Deep-Littré adds.
 
Citations are in their own table, joinable by `sense_id`:
 
```sql
SELECT c.author, c.resolved_author, c.text_plain, c.reference
FROM citations c
JOIN senses s ON c.sense_id = s.sense_id
JOIN entries e ON s.entry_id = e.entry_id
WHERE e.headword = 'TRONQUER';
```
```
LA FONT.  | LA FONTAINE  | Il ôte de chez lui les branches …  | Fabl. XII, 20
BUFF.     | BUFFON       | Le coati est sujet à manger …      | Quadrup. t. III, p. 83
BOSSUET   | BOSSUET      | Ces auteurs tronquent le passage … | Var. XI, 112
…
```
 
Locutions have their own table:
 
```sql
SELECT l.canonical_form, s.content_plain
FROM locutions l
JOIN senses s ON l.sense_id = s.sense_id
JOIN entries e ON s.entry_id = e.entry_id
WHERE e.headword = 'TRONQUER';
```
```
En parlant des statues | En parlant des statues, mutiler en partie. …
```
 
 
## A harder case: ŒUVRE
Larger entries can have dozens of numbered senses, each branching into specialized uses, fixed expressions, and editorial commentary. To see the variety, consider this fragment from the entry for ŒUVRE, sense 12.
 
Sense 12 defines _œuvre_ as "toute sorte d'actions morales" (any kind of moral action). Under it, Littré narrows the focus with the grammatical note _Absolument_, meaning: when _les œuvres_ is used on its own, without a complement, it refers specifically to meritorious actions. This is what Deep-Littré calls a transition group: a grammatical label that scopes over everything beneath it.
 
```xml
<sense xml:id="uvre_s12.3">
  <usg type="gram">absolument</usg>
  <def>. Les œuvres, les actions méritoires.</def>
  [… citations from Bossuet, Saci omitted …]
```
 
Under that transition there are eight sub-entries, each doing something quite different:
 
An **elaboration** narrows the parent sense:
```xml
<sense xml:id="uvre_s12.3.1">
  <def>
    Particulièrement, dans saint Paul, les œuvres, les actes
    rigoureusement conformes à la loi et les pratiques
    minutieuses que les Juifs y rattachaient
  </def>
</sense>
```
"Particulièrement" here is an explicit narrowing word. This isn't a new fixed expression; it's specifying what _les œuvres_ means in a particular author's usage.

A **continuation** extends or supplements the parent sense:
```xml
<sense xml:id="uvre_s3.3">
    <def>On dit aussi en ce sens : exécuteur des hautes œuvres.</def>
    <cit type="example">
      <quote>
        Dans la vue d'inspirer plus d'horreur pour le crime, l'entrée 
        de la ville est interdite à l'exécuteur des hautes œuvres
      </quote>
      <bibl>
        <author>BARTHÉL.</author>
        <biblScope>Anach. ch. 73</biblScope>
      </bibl>
    </cit>
</sense>
```
 
A **locution** defines a fixed expression:
```xml
<sense xml:id="uvre_s12.3.2">
  <def>
    Œuvres de miséricorde, celles qui ont pour objet
    la charité envers le prochain.
  </def>
</sense>
```
This has the classic _form-comma-gloss_ pattern: _Œuvres de miséricorde_ is the fixed phrase (_form_), and everything after the comma is its definition (_gloss_).
 
More locutions:
```xml
<sense xml:id="uvre_s12.3.4">
  <def>
    Œuvres pies, les œuvres que la piété fait faire,
    que l'on fait en vue de Dieu.
  </def>
</sense>
```
 
```xml
<sense xml:id="uvre_s12.3.5">
  <def>
    Œuvres de surérogation, celles auxquelles on n'est
    pas obligé par le précepte ou le devoir.
  </def>
</sense>
```
 
Another elaboration — editorial commentary on a previously defined expression:
```xml
<sense xml:id="uvre_s12.3.6">
  <def>
    Dans le langage général, œuvre de surérogation se dit
    de tout ce qu'on fait au delà du devoir, au delà de
    ce qui est nécessaire à l'affaire dont il s'agit.
  </def>
</sense>
```
 
This extends _œuvre de surérogation_ from its religious sense into general usage. It's commentary, not a new definition.
 
The difficulty is that all of these are encoded in the source XML under the tag `<indent>`. A locution and an elaboration may sit side by side with no markup to distinguish them. Even the text-level signals are subtle: the difference between "Œuvres de miséricorde, celles qui..." (a locution defining a fixed phrase) and "Particulièrement, dans saint Paul, les œuvres..." (an elaboration narrowing a sense) is a judgment call that requires understanding what Littré is doing, not just pattern-matching on punctuation.
 
This is the core challenge of the Deep-Littré project.
 
## Work in progress
Deep-Littré is an ongoing project. The pipeline produces usable data as-is, but the structural classification is still being refined. Improvements to the classification pipeline will be the subject of future posts.
 
The code and data are available at [github.com/myersm0/deep-littre](https://github.com/myersm0/deep-littre).
 



