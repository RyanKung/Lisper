---
name: lisper
description: Strict Common Lisp and Emacs Lisp review and generation for warning-free compilation, structured conditions, hygienic macros, pure functional cores, lexical-binding discipline, semantic predicates, documented symbols, and non-destructive list semantics. Use when Common Lisp or Elisp work needs model completeness, macro hygiene, condition/restart design, buffer/process boundary isolation, checkdoc/package-lint/sblint compliance, and bounded code size.
---

# Lisper

## Operating Rule

Use this skill as a set of assertions. Treat each assertion as satisfied, violated, or explicitly deferred by a user constraint. Prefer exact code facts over taste claims. Name the package, symbol, struct, condition, macro expansion, invariant, test, or side-effect boundary that makes the assertion true.

State the fact. Name the witness. Do not argue taste.

When reporting about Common Lisp or Emacs Lisp code, write in short declarative statements:

- The invariant is in the constructor.
- The dynamic extent is bounded by `unwind-protect`.
- The macro leaks evaluation order.
- The condition is structured.
- The destructive list op mutates caller structure.
- The predicate is naked.
- The compiler warning remains.
- The buffer effect leaks into the pure core.

## Assertions

1. Code pictures a model. S-expressions are syntax trees before they are text. Preserve every real state of the problem. Use symbols, tagged structures (`defstruct` / `cl-defstruct`), CLOS classes, or algebraic alists/plists with explicit constructors. Erase only distinctions that are not semantically used.

2. Higher-order functions compose before macros rewrite. Lisp is lambda calculus first: functions are first-class values, closures carry lexical environments, and currying (`apply-partially` or returning `(lambda (x) ...)`) replaces multi-parameter plumbing. When the same structure appears twice, inspect it; when it appears three times, abstract it with a higher-order combinator, curried closure, or generic function—never a 10-argument procedural helper. Introduce a macro only to control evaluation, bind lexical symbols, or eliminate compile-time boilerplate. Every macro must evaluate value arguments at most once in left-to-right order and bind temporaries with `gensym` / `make-symbol` (`with-gensyms`).

3. Pure data transforms stay outside the image's mutable world. Prefer catamorphisms (`cl-reduce` / `seq-reduce`), homomorphic mappings (`mapcar` / `seq-keep`), and predicate pipelines (`seq-filter`) over imperative `let` + `setq` + `push` + `nreverse` loops. Separate pure symbolic/list/structure transformations from buffer mutation, process IO, filesystem state, network streams, timers, and global specials (`*earmuffs*`). Core functions take values and return values. Host effects stay in named boundary functions.

4. Lexical scope is the default law. Every Emacs Lisp file must begin with `;; -*- lexical-binding: t; -*-` on line 1. Every dynamic special variable in Common Lisp or Emacs Lisp must wear `*earmuffs*` (or `+plus-signs+` for constants in CL) and be rebound dynamically with `let` rather than mutated globally with `setq` / `setf`, unless changing global configuration is the explicit contract.

5. Invariants are propositions. Express invariants in types (`deftype` with `satisfies`, `ftype` proclamations, `defstruct` `:type` slots) first, smart constructors (`:constructor`) second, runtime assertions (`check-type` / `cl-check-type`) third, tests fourth, and comments last. A constructor is a proof that a structure satisfies its slot invariants.

6. Conditions are algebraic facts, not raw strings. Define explicit condition hierarchies (`define-condition` in Common Lisp; `define-error` in Emacs Lisp). Signal structured conditions with `error` / `cerror` / `signal` so callers can handle them with `handler-case`, `handler-bind`, or `condition-case` without matching error message substrings. Use Common Lisp restarts (`restart-case`) when recovery policy belongs to the caller and mechanism belongs to the low-level function.

7. Cons-cell ownership is a semantic claim. Treat input lists, trees, and sequences as immutable unless the function's name and docstring explicitly declare destructive behavior (`nreverse`, `nconc`, `delete`, `sort`, `cl-delete`, `setf` on shared cons cells). Never apply destructive operations to quoted literals (`'(a b c)`) or borrowed caller lists. Duplicate structure (`copy-list`, `copy-tree`, `copy-sequence`) only at the boundary where mutation begins.

8. Quality is witnessed, not asserted. Zero byte-compiler warnings, zero `checkdoc` / `package-lint` errors, and zero SBCL `STYLE-WARNING` diagnostics are only the floor. Add ERT (`ert-deftest`) or FiveAM/Rove/Parachute tests for pure transformations, macro-expansion laws, boundary inputs, dynamic cleanup under non-local exit, and fixed regressions. A test name states the proposition it witnesses.

9. Documentation and namespace prefixes are part of the public contract.
   - In **Common Lisp**: define explicit packages (`defpackage` / `uiop:define-package`) without wholesale `:use` pollution beyond `:cl`; document every exported function, macro, class, variable, and condition.
   - In **Emacs Lisp**: prefix every public symbol with `<package>-` and every internal symbol with `<package>--`; write `checkdoc`-compliant docstrings whose first line is a complete sentence and whose parameter references are `UPPERCASE`; declare `Package-Requires` and `;;; Commentary:` headers accurately.

10. Resource lifetime is bounded by dynamic extent. Never pair manual allocation and cleanup across fallible expressions. Wrap every temporary buffer change, point movement, file handle, process, socket, or hook installation in `unwind-protect` (or `with-open-file`, `with-temp-buffer`, `save-excursion`, `save-restriction`, `save-match-data`). Non-local exits (`throw`, `return-from`, `signal`, `error`) must leave global state and buffers clean.

11. Size and parenthesis layout are structural signals. Keep each source file under 1000 lines and each `defun` / `defmacro` under 80 lines. Never place closing parentheses `)))` alone on trailing lines. Indent according to standard Lisp form semantics (declaring `(declare (indent ...))` on every Elisp macro). Split files by domain proposition, not vague `utils` buckets.

12. Logic deserves a name. Do not expose raw `car`/`cdr`/`nth` chains (`cadddr`), nested `assoc`/`plist-get` plumbing, or multi-clause `and`/`or`/`not` expressions at the call site when they express a domain proposition. Define named accessors and semantic predicates ending in `-p` or `p` (e.g., `order-matchable-p`, `same-principal-p`, `session-active-p`).

## Model Completeness

Before choosing packages, structs, and functions, name the formal model that makes the code coherent.

For macros and syntactic transformations:

- A macro is a compiler pass from syntax to syntax: $\text{AST}_{\text{surface}} \to \text{AST}_{\text{core}}$.
- Specify the surface grammar (destructuring lambda list), the binding discipline, the evaluation multiplicity of every subform, and the target expansion.
- Verify hygiene with `macroexpand-1` / `macrostep`: no free symbol capture, no multiple evaluation of effectful arguments, and no runtime `eval`.
- If a macro expands into definitions, preserve source locations and declarations (`declare`, docstring placement) for the host compiler and inspector.

For interactive state, buffers, and concurrent processes:

- An interactive system is a state transition relation before it is a buffer or thread.
- Separate the domain state value from the rendering buffer (Emacs UI) or worker thread/socket (CL/Emacs process).
- Model asynchronous process output (Emacs process filters/sentinels, CL threads/channels) as discrete events feeding a total state transition: $\text{State} \times \text{Event} \to (\text{State} \times \text{List}[\text{Effect}])$.
- Process filters in Emacs Lisp may receive arbitrary chunk boundaries; buffer and parse incomplete frames explicitly before dispatching domain events.
- Guard buffer-local state (`defvar-local`) against killed buffers (`buffer-live-p`) when timers, sentinels, or callbacks fire asynchronously.

For conditions, restarts, and non-local control:

- Separate **what went wrong** (the condition datum) from **how to recover** (restarts in CL; structured handlers in Elisp) and **which recovery to choose** (the outer policy handler).
- Never swallow conditions with a bare `(ignore-errors ...)` or `(condition-case nil ... (error nil))` unless the exact ignored condition type is narrowed and the fallback value is a valid domain state.

## Formal Logic Comments

Use formal comments only when they clarify a proof obligation that `ftype`, `deftype`, or `cl-defstruct` does not fully express. Use `;;` inside functions/forms and `;;;` at top level, following standard Lisp comment hierarchy (`;;;;` section, `;;;` top-level, `;;` aligned in-body, `;` margin).

```lisp
;;; Pre: (and ( <= 0 start end (length sequence)) (listp sequence))
;;; Post: (and (equal result (subseq sequence start end))
;;;            (not (eq result sequence)))
(defun slice-copy-checked (sequence start end)
  "Return a fresh subsequence of SEQUENCE from START to END."
  (check-type sequence list)
  (assert (<= 0 start end (length sequence)))
  (copy-list (subseq sequence start end)))

;;; Invariant: (null (intersection (work-set-pending ws) (work-set-completed ws) :test #'equal))
;;; Preservation: `work-set-complete' removes ID from `pending' before consing onto `completed'.
(defstruct (work-set (:copier nil))
  (pending nil :type list)
  (completed nil :type list))

;;; Law: (equal (decode-packet (encode-packet x)) x) for all valid packet structs X.
```

Prefer these forms:

- `Pre`: the condition a caller must satisfy.
- `Post`: the condition the function guarantees, including freshness/sharing (`eq` vs `equal`).
- `Invariant`: the proposition preserved across struct/buffer/session operations.
- `Law`: an algebraic or macro-expansion relation that tests must witness.
- `Preservation`: why a transition or `unwind-protect` form keeps the invariant true.

## Semantic Predicates

Use named predicates following Lisp `-p` / `p` conventions to express domain logic declaratively:

- Single-word root (Common Lisp convention): `evenp`, `Orderp` -> `orderp`, `validp` (or in Elisp, `-p` consistently for package-prefixed symbols).
- Hyphenated root (both CL and Elisp): `order-matchable-p`, `same-principal-p`, `lispm-session-live-p`.

Prefer:

```lisp
(when (order-matchable-p order resting-order)
  (match-order book order resting-order))

(when (same-principal-p actor owner)
  (grant-access actor resource))
```

Avoid:

```lisp
(when (and (not (eq (order-side order) (order-side resting-order)))
           (>= (order-price order) (order-price resting-order))
           (plusp (order-quantity order)))
  (match-order book order resting-order))

(when (and (equal (principal-id actor) (principal-id owner))
           (eq (principal-realm actor) (principal-realm owner)))
  (grant-access actor resource))
```

Also avoid deep positional list access (`(cadddr record)`, `(nth 4 item)`) in domain logic; define a `defstruct` / `cl-defstruct` or named accessor functions so the call site reads the domain concept rather than the cons topology.

## Implementation Discipline

Start from the domain and respect the dialect's functional semantics.

- **Currying, Partial Application, and Combinators**:
  - Never decompose a long function by extracting a procedural helper with 7+ positional parameters that mirror the caller's `let*` bindings.
  - Instead, curry invariant context into lexical closures (`(defun domain--matcher (query role) (lambda (event) ...))` or `(apply-partially #'fn fixed-arg)`) and compose small unary/binary transformations: $f \circ g = \lambda x.\, f(g(x))$.
  - Replace repetitive imperative polling, retry, or resource-bracket patterns with higher-order combinators that accept a thunk or step function `(funcall step item)` and return values.
  - Express multi-branch syntax or protocol dispatch via declarative alist/plist dispatch tables mapping symbols to functions rather than sprawling `cond` ladders.
- **Equality precision**: Never use `eq` where `eql` (numbers/characters), `equal` (structural strings/lists), or `equalp` / domain equality is required. Choose the tightest equality predicate that matches the type's equivalence relation, and pass `:test #'equal` explicitly to `assoc`, `member`, `gethash`, `remove`, and `cl-find` when keys are strings or composite values.
- **Emacs Lisp specifics**:
  - Line 1 must declare `;; -*- lexical-binding: t; -*-`.
  - Use `cl-lib` (`cl-defstruct`, `cl-loop`, `cl-destructuring-bind`, `cl-check-type`), never the deprecated runtime `cl` package.
  - Guard all point, mark, narrowing, and match-data side effects with `save-excursion`, `save-restriction`, and `save-match-data`.
  - Prefer buffer-passing or explicit `(with-current-buffer buf ...)` over assuming `current-buffer` across asynchronous boundaries.
  - Annotate macros with `(declare (indent ...) (debug ...))` so indentation and Edebug instrumentation work automatically.
- **Common Lisp specifics**:
  - Proclaim `(declaim (ftype (function (...) ...) name))` for public domain functions so SBCL performs compile-time type inference and exhaustiveness checks.
  - Use `etypecase` / `ecase` / `ctypecase` instead of open-ended `case` / `typecase` so unhandled variants signal a structured type error immediately.
  - Avoid reader-table pollution and unexported symbol leaks (`pkg::internal-sym`).
  - Prefer ASDF package-inferred or explicit serial `:components` with clean dependency DAGs.
- **Forbidden constructs in production paths**:
  - No runtime `eval` to dispatch functions or construct code (use `funcall`, `apply`, or compile-time macros).
  - No dynamic `setq` on undeclared free variables.
  - No in-place mutation (`nconc`, `nreverse`, `delete`, `sort`) of quoted literals or unowned caller lists.
  - No bare `(ignore-errors ...)` hiding unexpected program bugs.
  - No dangling closing parentheses on their own lines.

## Review Protocol

When reviewing, writing, or refactoring Lisp code under this skill, run the verification tools appropriate to the dialect before claiming compliance. Always enforce an explicit command timeout.

### Emacs Lisp Verification Floor

```bash
# 1. Strict Byte-Compilation (warnings treated as errors) in clean Emacs
/opt/homebrew/bin/timeout 30 emacs -Q --batch \
  -L . \
  --eval "(setq byte-compile-error-on-warn t)" \
  -f batch-byte-compile <files.el>

# 2. Checkdoc & Package-Lint (when available or via Eask/Eldev)
/opt/homebrew/bin/timeout 30 emacs -Q --batch \
  --eval "(require 'checkdoc)" \
  --eval "(dolist (f command-line-args-left) (checkdoc-file f))" <files.el>

# 3. ERT Batch Test Suite
/opt/homebrew/bin/timeout 30 emacs -Q --batch \
  -L . -L test \
  -l <test-file.el> \
  -f ert-run-tests-batch-and-exit
```

### Common Lisp Verification Floor

```bash
# 1. Strict SBCL Compilation (fail on any WARNING or STYLE-WARNING)
/opt/homebrew/bin/timeout 60 sbcl --noinform --non-interactive \
  --eval '(require :asdf)' \
  --eval '(handler-bind ((warning (lambda (c) (uiop:die 1 "Compiler warning: ~A" c))))
            (asdf:compile-system :<system-name> :force t))'

# 2. sblint / lisp-critic (if installed in environment)
/opt/homebrew/bin/timeout 60 sblint <system.asd>

# 3. ASDF Test Suite
/opt/homebrew/bin/timeout 60 sbcl --noinform --non-interactive \
  --eval '(require :asdf)' \
  --eval '(asdf:test-system :<system-name>)'
```

Check structure:

- Every `.el` file has `lexical-binding: t` on line 1, valid package headers, and `provide` at the bottom.
- Every `.lisp` file begins with `(in-package #:...)` and belongs to an explicit ASDF component.
- Compiler output has zero `WARNING` and zero `STYLE-WARNING` diagnostics.
- Public symbols have contract-focused docstrings (`checkdoc`-clean in Elisp).
- Conditions use `define-condition` (CL) or `define-error` (Elisp), not ad-hoc strings.
- Macros are hygienic (`gensym`), single-evaluating, and necessary.
- Destructive sequence operations never mutate shared or literal structure.
- Equality predicates (`eq`, `eql`, `equal`) and hash/list `:test` arguments match key types.
- Dynamic state, buffers, files, and processes are protected by `unwind-protect` or `with-*` macros.
- Domain propositions are expressed as `-p` / `p` semantic predicates instead of naked boolean/cons plumbing.
- Closing parentheses are stacked on the final expression line, never dangling on separate lines.
- No source file exceeds 1000 lines; no `defun` / `defmacro` exceeds 80 lines.

## Response Pattern

When applying the skill, report findings as assertions:

```text
The file header omits lexical-binding: t in parser.el.
The macro with-retry evaluates the form argument twice without gensym.
The destructive sort in rank-candidates mutates the caller's list in place.
The assoc lookup on string keys omits :test #'equal.
The buffer excursion in render-view lacks save-excursion and unwind-protect.
The error in decode-frame signals an untyped string instead of a defined condition.
The cadddr chain in extract-header exposes tuple position instead of a struct accessor.
The conditional in can-dispatch-p is naked at the call site.
```

If a user asks for implementation, fix the violation directly. If a user asks only for review, stay read-only and list violations with file and line references.
