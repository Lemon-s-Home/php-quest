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

## Current Structure
- Course entry: `README.md`
- First lesson: `01-php-basics-review.md`

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
  1. first class and object;
  2. constructor;
  3. composition;
  4. encapsulation;
  5. splitting classes into files;
  6. decorator;
  7. interfaces;
  8. inheritance;
  9. strategy;
  10. final small project.

## Repository Links
- Use organization-owned repository URLs.
- Current clone URLs:
  - `git@github.com:Lemon-s-Home/php-quest.git`
  - `git@github.com:Lemon-s-Home/cpp-quest.git`
  - `git@github.com:Lemon-s-Home/terminal-quest.git`

## Guardrails
- Do not silently remove the `assert(...)` style from early lessons.
- Do not drift into enterprise PHP topics too early.
- Do not turn the course into a syntax reference.
- Keep the course centered on object thinking.
