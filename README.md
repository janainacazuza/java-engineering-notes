# java-engineering-notes

Technical notes on Java software engineering, organized by the criterion of direct impact on professional practice.

This is not a course, not a tutorial, and not a book summary. It is a living reference by a Java software engineer in formation, filtered by the question:

> "Would I write or design code differently if I knew this, even months from now?"

What does not pass that filter does not enter. What passes enters with sufficient depth to be actionable.

The vault systematically covers Java 21 (LTS) and Java 25 (LTS). When behavior, API, or trade-offs differ meaningfully between the two versions, the comparison is explicit in the note. Examples and production signals are anchored in financial systems engineering context when natural, without forcing the framing on general topics.

## How to use

Clone the repository and open the `vault/` folder in [Obsidian](https://obsidian.md):

```bash
git clone https://github.com/janainacazuza/java-engineering-notes.git
```

In Obsidian, choose **Open folder as vault** and select the `vault/` directory inside the cloned repository. Start at `vault/MOC.md`.

## Vault structure

````
vault/
├── _meta/                Editorial system (filter, template, graph, maintenance)
├── 10_mechanisms/        How the Java platform works internally
├── 20_decisions/         Trade-offs and selection criteria
├── 30_pitfalls/          Bugs the industry has paid dearly to learn
├── 40_patterns/          Forms with strict applicability criteria
├── 50_diagnostics/       What to do when something breaks in production
├── 60_code/              Canonical snippets and anti-snippets
├── 70_interview/         Typical questions and model answers
├── 99_inbox/             Quarantine for notes pending filter review
├── hubs/                 Topic field views, created when areas reach 5 reviewed notes
└── MOC.md                Map of content, the entry point of the vault
````

The full editorial rules are in `vault/_meta/`.

## About the project

Maintained by [Janaína Cazuza](https://github.com/janainacazuza). Notes reflect ongoing learning in the Java ecosystem with emphasis on engineering fundamentals that remain relevant across versions and frameworks.

The project follows a closed curatorial model to preserve editorial coherence. See [CONTRIBUTING.md](CONTRIBUTING.md) for community contribution channels.

The designated technical reviewer is [Higor Cazuza](https://github.com/cazuzahigor). Reviews are exchanged via issues in this repository.

## License

[CC BY-SA 4.0](LICENSE). You may use, adapt, and redistribute with attribution and under the same license.

Citation: Janaína Cazuza, https://github.com/janainacazuza/java-engineering-notes
