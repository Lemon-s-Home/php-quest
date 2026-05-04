# CLAUDE

## Repository Purpose
- This repository contains `PHP Quest`, a course on object-oriented programming in PHP.
- The course is a follow-up to earlier study of basic PHP, `terminal-quest`, and `cpp-quest`.
- The goal is not "PHP from zero", but a transition into proper OOP through PHP.

## Audience
- Primary student: the author's son.
- He already knows basic PHP:
  - variables;
  - strings;
  - arrays;
  - conditions and loops;
  - functions.
- He also already touched OOP ideas in C++:
  - classes;
  - composition;
  - decorators;
  - partial intro before strategy.
- C++ itself turned out to be too noisy and frustrating for continued OOP study.

## Course Direction
- Keep the same conceptual path as `cpp-quest`, but teach it in PHP.
- Focus on proper OOP:
  - small objects with behavior;
  - composition before inheritance;
  - decorators;
  - interfaces;
  - strategy.
- Use short game-themed examples and tasks.
- Optimize for clarity of ideas, not language cleverness.
- After strategy, pivot from isolated examples to a growing text-mode turn-based game.
- The student is currently interested in Telegram-like turn-based text games.
- The post-strategy infrastructure arc is intentional:
  1. namespace;
  2. custom autoload;
  3. Composer PSR-4 autoload plus VarDumper;
  4. PsySH;
  5. CLImate;
  6. PHP config for game balance numbers.
- Do not restore the old "files and state" mini-project as lesson 12; that was a `cpp-quest` carry-over and is not the right next step here.

## Current Structure
- Course entry: `README.md`
- Lessons written: 01–17
  - `01-php-basics-review.md` — recap: variables, strings, arrays, functions
  - `02-classes-and-objects.md` — first class and object
  - `03-objects-are-born-ready.md` — constructor, promoted properties
  - `04-composition.md` — composition: object owns object
  - `05-encapsulation.md` — encapsulation, getters without `get`, `__toString`
  - `06-files-and-arrays.md` — one class per file, `require_once`, Composer mention, arrays of objects
  - `07-interfaces.md` — interface, implements, polymorphism
  - `08-decorator.md` — decorator pattern on weapon
  - `09-inheritance.md` — extends, abstract class, abstract methods
  - `10-character-decorator.md` — decorator on character, inheritance trap, composition argument
  - `11-strategy.md` — strategy pattern, AttackStyle
  - `12-namespace.md` — namespace, full class names, `use`, still with `require_once`
  - `13-autoload.md` — custom `spl_autoload_register`
  - `14-composer-vardumper.md` — Composer PSR-4 autoload, `symfony/var-dumper`, `require-dev`
  - `15-psysh.md` — PsySH REPL for inspecting game objects
  - `16-climate.md` — `league/climate`, terminal output/input, `GameOutput`, `PlayerInput`
  - `17-config-numbers.md` — PHP config for weapon and effect numbers, no factory yet

## Lesson 01 Decisions
- Lesson 01 is a recap after C++, not a beginner intro to PHP from scratch.
- The intro should start with a short joke, then move into the contrast with C++.
- The current opening tone matters and should be preserved unless there is a good reason to rewrite it.
- The practical block should use functions plus `assert(...)`, not "just print something".
- Each task should have:
  - a clear function signature;
  - a short statement;
  - a ready `assert(...)` block.
- This is intentional preparation for later testing habits.

## Lesson Authoring Style
- Write lessons in Russian unless there is a specific reason not to.
- Keep the tone direct, compact, and readable.
- Avoid bloated explanations and avoid literally repeating the user's phrasing when rewriting text.
- Prefer one good example over several weaker ones.
- Keep humor sparse and placed deliberately.
- When humor is used, it should sound like an internal joke, not a stand-up routine.
- In introductions, avoid duplicated meaning across adjacent sentences.
- Explain the concrete pain/benefit, not abstract "PHP is easier" claims.

## Task Style
- Prefer tasks shaped around functions first.
- Use `assert(...)` in almost every early lesson.
- A good task usually checks one focused idea:
  - string building;
  - array transformation or summary;
  - a simple function with a branch.
- Keep tasks short enough to finish in one sitting.

## OOP Progression
- After recap, move quickly into objects.
- The expected path is:
  1. ✅ first class and object (lesson 02);
  2. ✅ constructor (lesson 03);
  3. ✅ composition (lesson 04);
  4. ✅ encapsulation (lesson 05);
  5. ✅ splitting classes into files (lesson 06);
  6. ✅ interfaces (lesson 07);
  7. ✅ decorator on weapon (lesson 08);
  8. ✅ inheritance and abstract classes (lesson 09);
  9. ✅ decorator on character, inheritance critique (lesson 10);
  10. ✅ strategy (lesson 11);
  11. ✅ namespace (lesson 12);
  12. ✅ custom autoload (lesson 13);
  13. ✅ Composer autoload and VarDumper (lesson 14);
  14. ✅ PsySH (lesson 15);
  15. ✅ CLImate for terminal I/O (lesson 16);
  16. ✅ PHP config for weapon and effect numbers, without factories (lesson 17);
  17. continue growing the text-mode game (planned).

## Inheritance Teaching Plan
- Do not present inheritance only as a bad idea from the start.
- First let it look reasonable on a small game example:
  - `Character`;
  - `Warrior`;
  - `Mage`;
  - `Archer`.
- Then expand the same domain with new requirements that put pressure on the hierarchy:
  - temporary effects;
  - combinations of buffs or states;
  - equipment changes;
  - behaviors that should vary without turning into "new creature types".
- The student should feel the code becoming awkward before the course explains why.
- After that, revisit the same kind of problem through composition and show why it scales better.
- Do not call this sequence a "trap" in repository text or lesson text.

## Repository Links
- Use organization-owned repository URLs.
- Current clone URLs:
  - `git@github.com:Lemon-s-Home/php-quest.git`
  - `git@github.com:Lemon-s-Home/cpp-quest.git`
  - `git@github.com:Lemon-s-Home/terminal-quest.git`

## Haspadar Workflow
- Main workflow state: `docs/roadmap.md`
- External subroadmaps: `docs/roadmaps/`
- Research summaries: `docs/research/`
- Decisions: `docs/decisions/`
- Fast CLI: `chef status`, `chef next`, `chef next <number>`, `chef roadmap`, `chef add --next/--end`, `chef diff`, `chef projects`, `chef docker`
- Use `/chef next` or `chef next` to recover current state and continue work.
- Roadmap/status output shows transient numeric selectors by default; use `chef next <number>` or `chef next #<number>` to work a specific item in parallel sessions, and `--no-number` for parser-friendly output.
- Docker sandbox: `chef docker <project> codex|claude` runs agents inside the shared Chef container; the chef skill owns mounts for `~/.ssh`, git config, `~/.config/gh`, `~/.claude`, `~/.codex`, host Docker socket passthrough for sibling containers, forwards `SSH_AUTH_SOCK` when available, and passes host `GH_TOKEN`/host `gh auth token` ephemerally for GitHub HTTPS auth and SSH-origin-to-HTTPS Git URL rewrite. Use `chef docker <project> doctor` for smoke diagnostics.

## Code Style Decisions
- Methods are always expanded (never one-liners), including in full examples.
- PHPDoc comments (`/** ... */`) go on classes in composite examples (multiple classes together). Not on individual methods, not in short illustrative snippets.
- Interface names: use domain nouns (`Weapon`, `Fightable`), not `-er` suffixes (`Striker`).
- Concrete classes use specific names (`Sword`, `Axe`, `Dagger`) instead of generic base names (`BaseWeapon`).
- Decorator wrappers are named after the effect, not the type (`Poisoned`, `Fire`, `Frost`, not `PoisonedWeapon`, `FireWeapon`).
- Getters are named after the property without `get` prefix (`hp()`, `name()`), because PHP allows method and property to share a name.
- Each lesson has exactly 2 tasks (not 3): first simpler, second harder.
- Tasks must not repeat examples from the lesson — they should check a new angle of the same idea.
- `Напоследок` section contains only curious, funny, or historically interesting facts — no lesson summaries or previews of the next lesson.
- Post-Composer examples use namespace root `Game\`, not `PhpQuest\` or vendor-style organization names.
- First external packages in the course are:
  - `symfony/var-dumper` as `require-dev`;
  - `psy/psysh` as `require-dev`;
  - `league/climate` as a runtime dependency.
- Avoid `symfony/console` for now; it is too framework-shaped for this stage and can come later if the game needs CLI commands.
- Config lessons should use PHP config files first, e.g. `config/weapons.php` returning an array. Do not discuss JSON until file parsing/error handling itself becomes the topic.
- Lesson 17 intentionally avoids factories: it only moves balance numbers into PHP config while `main.php` still assembles objects manually.
- Do not introduce static factories, enums, or a `FromConfig implements Weapon` decorator for this stage.
- Keep the distinction clear:
  - code stores behavior;
  - config stores balance numbers first; names, ids, and effect lists can come later when object creation becomes the topic;
  - future save files store player-specific state such as equipped weapon id, HP, inventory, active effects.

## Guardrails
- Do not silently remove the `assert(...)` style from early lessons.
- Do not drift into enterprise PHP topics too early.
- Do not turn the course into a syntax reference.
- Keep the course centered on object thinking.
- Keep Composer/tooling lessons tied to the student's game project pain: many classes, namespaces, autoload, object inspection, terminal I/O.
