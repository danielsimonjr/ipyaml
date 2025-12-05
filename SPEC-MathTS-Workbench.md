# MathTS Scientific Workbench Specification

## YAML-Based Computational Notebook System

**Version:** 1.0.0-draft
**Date:** December 2024
**Status:** Specification / Product Description

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Vision & Goals](#2-vision--goals)
3. [Current State Analysis](#3-current-state-analysis)
4. [Target Architecture](#4-target-architecture)
5. [MathTS Integration Layer](#5-mathts-integration-layer)
6. [YAML Workbook Format Specification](#6-yaml-workbook-format-specification)
7. [Core Components](#7-core-components)
8. [API Design](#8-api-design)
9. [Feature Roadmap](#9-feature-roadmap)
10. [Technical Requirements](#10-technical-requirements)
11. [Migration Strategy](#11-migration-strategy)

---

## 1. Executive Summary

### Project Overview

**MathTS Workbench** is a TypeScript-based scientific computation platform that combines:

- **ipyaml's** proven YAML ↔ Jupyter notebook conversion capabilities
- **Math.js** mathematical computing power wrapped in a TypeScript-native interface (**MathTS**)
- **Maple/Mathematica-style** document-centric computational workflow

The result is a modern, type-safe scientific workbench where mathematical documents are human-readable YAML files with embedded symbolic computation, unit handling, and reproducible results.

### Key Value Propositions

| Stakeholder | Value |
|-------------|-------|
| **Scientists/Engineers** | Maple-style worksheets in version-controllable YAML format |
| **Developers** | Type-safe mathematical computing with full IDE support |
| **Educators** | Literate programming documents with live computation |
| **Data Teams** | Reproducible, diffable computational notebooks |

### Project Name

**`mathts-workbench`** (npm package)
Subpackages:
- `@mathts/core` - MathTS mathematical engine wrapper
- `@mathts/workbook` - YAML workbook format and runtime
- `@mathts/cli` - Command-line interface
- `@mathts/jupyter` - Jupyter/IPython interoperability

---

## 2. Vision & Goals

### Vision Statement

> Create the definitive open-source scientific workbench format that is:
> - **Human-readable** (YAML, not JSON blobs)
> - **Version-control friendly** (diffable, mergeable)
> - **Computationally powerful** (symbolic math, units, matrices)
> - **Type-safe** (TypeScript throughout)
> - **Interoperable** (Jupyter, LaTeX, Markdown export)

### Maple/Mathematica Paradigm

Drawing inspiration from commercial scientific workbenches:

```
┌─────────────────────────────────────────────────────────────┐
│  Maple Workbook Paradigm                                    │
├─────────────────────────────────────────────────────────────┤
│  • Document = Sequence of executable regions                │
│  • Regions contain: text, math input, math output, plots    │
│  • Execution flows top-to-bottom with dependency tracking   │
│  • Variables persist across regions (session state)         │
│  • Rich symbolic computation (solve, simplify, expand)      │
│  • Unit-aware calculations                                  │
│  • Publication-quality output (LaTeX, PDF)                  │
└─────────────────────────────────────────────────────────────┘
```

### Primary Goals

1. **Complete TypeScript Rewrite** - Port ipyaml from Python to TypeScript
2. **MathTS Engine** - Type-safe Math.js wrapper with enhanced API
3. **Scientific Workbook Format** - YAML schema for computational documents
4. **Execution Engine** - Runtime for evaluating workbook cells
5. **Bidirectional Jupyter Compatibility** - Import/export .ipynb files
6. **CLI Tooling** - Headless execution, conversion, validation

### Non-Goals (v1.0)

- GUI/IDE implementation (leave to downstream tools)
- Real-time collaboration features
- Cloud execution infrastructure
- Language kernels beyond TypeScript/JavaScript

---

## 3. Current State Analysis

### ipyaml Architecture (Python)

```
ipyaml/
├── convert.py          # Core: nb_to_yaml(), yaml_to_nb()
├── cli.py              # CLI: argparse-based interface
├── contents_manager.py # Jupyter: FileContentsManager extension
├── api.py              # Public exports
└── tests/              # pytest suite
```

**Strengths to Preserve:**
- Clean separation of concerns
- Bidirectional lossless conversion
- Human-readable YAML output format
- Jupyter integration pattern

**Limitations to Address:**
- No mathematical computation (pure format conversion)
- Python-only ecosystem
- No execution capabilities
- No type definitions
- Limited cell metadata support

### Current YAML Format

```yaml
cells:
  - markdown: |
      # Title
  - code: |
      print("Hello")
    id: 0
    metadata:
      collapsed: false

metadata:
  kernelspec:
    display_name: Python 3
    name: python3

data:
  - {execution_count: 1, outputs: [...]}
```

---

## 4. Target Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                     MathTS Workbench Architecture                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │   CLI       │    │   VS Code   │    │   Web UI    │  Consumers  │
│  │   Tools     │    │   Extension │    │   (future)  │             │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘             │
│         │                  │                  │                     │
│         └──────────────────┼──────────────────┘                     │
│                            │                                        │
│  ┌─────────────────────────┴─────────────────────────────┐         │
│  │              @mathts/workbook                          │         │
│  │  ┌─────────────┬─────────────┬─────────────────────┐  │         │
│  │  │   Parser    │  Executor   │   Serializer        │  │         │
│  │  │   (YAML)    │  (Runtime)  │   (YAML/ipynb)      │  │         │
│  │  └──────┬──────┴──────┬──────┴──────────┬──────────┘  │         │
│  └─────────┼─────────────┼─────────────────┼─────────────┘         │
│            │             │                 │                        │
│  ┌─────────┴─────────────┴─────────────────┴─────────────┐         │
│  │                    @mathts/core                        │         │
│  │  ┌─────────────┬─────────────┬─────────────────────┐  │         │
│  │  │  Symbolic   │   Units     │   Linear Algebra    │  │         │
│  │  │  Math       │   System    │   & Statistics      │  │         │
│  │  └─────────────┴─────────────┴─────────────────────┘  │         │
│  │                         │                              │         │
│  │              ┌──────────┴──────────┐                  │         │
│  │              │      Math.js        │                  │         │
│  │              │   (underlying lib)  │                  │         │
│  │              └─────────────────────┘                  │         │
│  └───────────────────────────────────────────────────────┘         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Package Structure

```
mathts-workbench/
├── packages/
│   ├── core/                    # @mathts/core
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── engine.ts        # MathTS engine wrapper
│   │   │   ├── types.ts         # Type definitions
│   │   │   ├── symbolic/        # Symbolic math operations
│   │   │   ├── units/           # Unit system
│   │   │   ├── matrix/          # Linear algebra
│   │   │   └── functions/       # Mathematical functions
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── workbook/                # @mathts/workbook
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── parser.ts        # YAML parsing
│   │   │   ├── serializer.ts    # YAML/ipynb output
│   │   │   ├── executor.ts      # Cell execution engine
│   │   │   ├── scope.ts         # Variable scope management
│   │   │   ├── types.ts         # Workbook type definitions
│   │   │   └── schema/          # JSON Schema for validation
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── cli/                     # @mathts/cli
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── commands/
│   │   │   │   ├── convert.ts   # Format conversion
│   │   │   │   ├── run.ts       # Execute workbook
│   │   │   │   ├── validate.ts  # Schema validation
│   │   │   │   └── export.ts    # LaTeX/Markdown export
│   │   │   └── utils.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── jupyter/                 # @mathts/jupyter
│       ├── src/
│       │   ├── index.ts
│       │   ├── notebook.ts      # nbformat types
│       │   ├── import.ts        # .ipynb → workbook
│       │   └── export.ts        # workbook → .ipynb
│       ├── package.json
│       └── tsconfig.json
│
├── package.json                 # Monorepo root
├── tsconfig.base.json           # Shared TS config
├── pnpm-workspace.yaml          # Workspace definition
└── turbo.json                   # Build orchestration
```

---

## 5. MathTS Integration Layer

### Design Philosophy

MathTS wraps Math.js with:
- **Full TypeScript type safety** - Generic types for all operations
- **Fluent API** - Chainable method calls
- **Enhanced error messages** - Domain-specific error handling
- **Serialization support** - YAML-friendly result types

### Core Types

```typescript
// @mathts/core/src/types.ts

/**
 * Base result type for all MathTS operations
 */
export interface MathResult<T = unknown> {
  value: T;
  type: ResultType;
  latex?: string;        // LaTeX representation
  display?: string;      // Human-readable display
  units?: UnitInfo;      // Unit information if applicable
  metadata?: ResultMetadata;
}

export type ResultType =
  | 'number'
  | 'complex'
  | 'fraction'
  | 'bignum'
  | 'matrix'
  | 'unit'
  | 'symbolic'
  | 'boolean'
  | 'string'
  | 'function'
  | 'undefined';

export interface UnitInfo {
  dimensions: Record<string, number>;
  baseUnit: string;
  displayUnit?: string;
}

export interface ResultMetadata {
  precision?: number;
  simplified?: boolean;
  evaluationTime?: number;
}

/**
 * Symbolic expression type
 */
export interface SymbolicExpression {
  expression: string;
  variables: string[];
  toLatex(): string;
  evaluate(scope: Record<string, number>): MathResult<number>;
  simplify(): SymbolicExpression;
  expand(): SymbolicExpression;
  differentiate(variable: string): SymbolicExpression;
  integrate(variable: string): SymbolicExpression;
}

/**
 * Matrix with full type information
 */
export interface TypedMatrix<T extends number | Complex = number> {
  data: T[][];
  rows: number;
  cols: number;
  determinant(): MathResult<T>;
  inverse(): TypedMatrix<T>;
  eigenvalues(): MathResult<T[]>;
  transpose(): TypedMatrix<T>;
}

/**
 * Unit-aware quantity
 */
export interface Quantity<U extends string = string> {
  value: number;
  unit: U;
  to<V extends string>(targetUnit: V): Quantity<V>;
  add(other: Quantity<U>): Quantity<U>;
  multiply(other: Quantity): Quantity;
}
```

### MathTS Engine API

```typescript
// @mathts/core/src/engine.ts

import { create, all, MathJsStatic } from 'mathjs';

export class MathTS {
  private math: MathJsStatic;
  private scope: Map<string, unknown>;

  constructor(config?: MathTSConfig) {
    this.math = create(all, config?.mathjs);
    this.scope = new Map();
  }

  // ═══════════════════════════════════════════════════════════════
  // EVALUATION
  // ═══════════════════════════════════════════════════════════════

  /**
   * Evaluate a mathematical expression
   * @example
   * math.evaluate('2 + 2') // => { value: 4, type: 'number' }
   * math.evaluate('x^2', { x: 3 }) // => { value: 9, type: 'number' }
   */
  evaluate(expression: string, scope?: Record<string, unknown>): MathResult {
    // Implementation
  }

  /**
   * Parse expression without evaluating (symbolic)
   */
  parse(expression: string): SymbolicExpression {
    // Implementation
  }

  // ═══════════════════════════════════════════════════════════════
  // SYMBOLIC OPERATIONS (Maple-style)
  // ═══════════════════════════════════════════════════════════════

  /**
   * Simplify an expression
   * @example
   * math.simplify('x^2 + 2*x + 1') // => (x + 1)^2
   */
  simplify(expression: string | SymbolicExpression): SymbolicExpression {
    // Implementation
  }

  /**
   * Expand an expression
   * @example
   * math.expand('(x + 1)^2') // => x^2 + 2*x + 1
   */
  expand(expression: string | SymbolicExpression): SymbolicExpression {
    // Implementation
  }

  /**
   * Solve equation(s) for variable(s)
   * @example
   * math.solve('x^2 - 4 = 0', 'x') // => [2, -2]
   * math.solve(['x + y = 10', 'x - y = 2'], ['x', 'y']) // => {x: 6, y: 4}
   */
  solve(
    equations: string | string[],
    variables: string | string[]
  ): MathResult<number | number[] | Record<string, number>> {
    // Implementation
  }

  /**
   * Differentiate expression with respect to variable
   * @example
   * math.diff('x^3 + 2*x', 'x') // => 3*x^2 + 2
   */
  diff(expression: string, variable: string, order?: number): SymbolicExpression {
    // Implementation
  }

  /**
   * Integrate expression with respect to variable
   * @example
   * math.integrate('3*x^2 + 2', 'x') // => x^3 + 2*x
   */
  integrate(
    expression: string,
    variable: string,
    bounds?: [number, number]
  ): SymbolicExpression | MathResult<number> {
    // Implementation
  }

  // ═══════════════════════════════════════════════════════════════
  // UNITS (Engineering-focused)
  // ═══════════════════════════════════════════════════════════════

  /**
   * Create a unit-aware quantity
   * @example
   * math.unit(100, 'km/h').to('m/s') // => 27.78 m/s
   */
  unit(value: number, unitStr: string): Quantity {
    // Implementation
  }

  /**
   * Define custom unit
   */
  defineUnit(name: string, definition: string | UnitDefinition): void {
    // Implementation
  }

  // ═══════════════════════════════════════════════════════════════
  // LINEAR ALGEBRA
  // ═══════════════════════════════════════════════════════════════

  /**
   * Create a matrix
   * @example
   * math.matrix([[1, 2], [3, 4]])
   */
  matrix<T extends number | Complex = number>(data: T[][]): TypedMatrix<T> {
    // Implementation
  }

  /**
   * Solve linear system Ax = b
   */
  linsolve(A: TypedMatrix, b: number[]): MathResult<number[]> {
    // Implementation
  }

  // ═══════════════════════════════════════════════════════════════
  // SCOPE MANAGEMENT
  // ═══════════════════════════════════════════════════════════════

  /**
   * Set a variable in the current scope
   */
  set(name: string, value: unknown): void {
    this.scope.set(name, value);
  }

  /**
   * Get a variable from scope
   */
  get(name: string): unknown {
    return this.scope.get(name);
  }

  /**
   * Clear all variables
   */
  clearScope(): void {
    this.scope.clear();
  }

  /**
   * Get current scope as serializable object
   */
  exportScope(): Record<string, unknown> {
    return Object.fromEntries(this.scope);
  }

  /**
   * Import scope from object
   */
  importScope(scope: Record<string, unknown>): void {
    Object.entries(scope).forEach(([k, v]) => this.scope.set(k, v));
  }
}

export interface MathTSConfig {
  mathjs?: Partial<ConfigOptions>;
  precision?: number;
  defaultUnits?: 'SI' | 'imperial' | 'CGS';
  autoSimplify?: boolean;
}
```

### Usage Examples

```typescript
import { MathTS } from '@mathts/core';

const math = new MathTS({ precision: 15 });

// Basic evaluation
const result = math.evaluate('sqrt(2) + e^(i*pi)');
console.log(result.value);   // 0.41421356...
console.log(result.latex);   // \sqrt{2} + e^{i\pi}

// Symbolic computation (Maple-style)
const expr = math.parse('x^2 + 2*x + 1');
const simplified = math.simplify(expr);
console.log(simplified.toLatex()); // (x + 1)^2

const derivative = math.diff('sin(x) * cos(x)', 'x');
console.log(derivative.expression); // cos(x)^2 - sin(x)^2

// Solving equations
const solutions = math.solve('x^2 - 5*x + 6 = 0', 'x');
console.log(solutions.value); // [2, 3]

// Units (engineering)
const speed = math.unit(100, 'km/h');
const inMps = speed.to('m/s');
console.log(inMps.value, inMps.unit); // 27.778 m/s

// Linear algebra
const A = math.matrix([[3, 1], [1, 2]]);
console.log(A.determinant().value); // 5
console.log(A.eigenvalues().value); // [3.618, 1.382]
```

---

## 6. YAML Workbook Format Specification

### Format Overview

The MathTS Workbook format (`.mtsw` or `.mathts.yaml`) is a human-readable, version-control-friendly format for scientific documents.

### File Extensions

| Extension | Description |
|-----------|-------------|
| `.mtsw` | MathTS Workbook (primary) |
| `.mathts.yaml` | Alternative explicit extension |
| `.mathts.yml` | Alternative short form |

### Schema Definition

```yaml
# MathTS Workbook Schema v1.0
# File: workbook.schema.yaml

$schema: "https://json-schema.org/draft/2020-12/schema"
$id: "https://mathts.dev/schemas/workbook/v1"
title: "MathTS Scientific Workbook"
type: object

required:
  - version
  - cells

properties:
  version:
    type: string
    pattern: "^1\\.\\d+$"
    description: "Workbook format version"

  metadata:
    $ref: "#/$defs/WorkbookMetadata"

  cells:
    type: array
    items:
      $ref: "#/$defs/Cell"

  data:
    type: object
    description: "Computed outputs and cached results"
    additionalProperties: true

$defs:
  WorkbookMetadata:
    type: object
    properties:
      title:
        type: string
      authors:
        type: array
        items:
          $ref: "#/$defs/Author"
      created:
        type: string
        format: date-time
      modified:
        type: string
        format: date-time
      description:
        type: string
      tags:
        type: array
        items:
          type: string
      kernel:
        $ref: "#/$defs/KernelSpec"
      settings:
        $ref: "#/$defs/WorkbookSettings"

  Author:
    type: object
    properties:
      name:
        type: string
      email:
        type: string
        format: email
      affiliation:
        type: string

  KernelSpec:
    type: object
    properties:
      name:
        type: string
        enum: [mathts, typescript, javascript, python]
      version:
        type: string

  WorkbookSettings:
    type: object
    properties:
      precision:
        type: integer
        minimum: 1
        maximum: 64
        default: 15
      unitSystem:
        type: string
        enum: [SI, imperial, CGS, natural]
        default: SI
      autoExecute:
        type: boolean
        default: false
      outputFormat:
        type: string
        enum: [latex, ascii, unicode]
        default: unicode

  Cell:
    oneOf:
      - $ref: "#/$defs/MarkdownCell"
      - $ref: "#/$defs/MathCell"
      - $ref: "#/$defs/CodeCell"
      - $ref: "#/$defs/PlotCell"
      - $ref: "#/$defs/TableCell"

  MarkdownCell:
    type: object
    required: [markdown]
    properties:
      markdown:
        type: string
      id:
        type: string
        format: uuid

  MathCell:
    type: object
    required: [math]
    properties:
      math:
        type: string
        description: "MathTS expression to evaluate"
      id:
        type: string
        format: uuid
      label:
        type: string
        description: "Variable name to store result"
      display:
        type: string
        enum: [inline, block, hidden]
        default: block
      output:
        $ref: "#/$defs/MathOutput"

  MathOutput:
    type: object
    properties:
      value:
        description: "Computed result value"
      latex:
        type: string
      display:
        type: string
      executionTime:
        type: number
      error:
        type: string

  CodeCell:
    type: object
    required: [code]
    properties:
      code:
        type: string
        description: "TypeScript/JavaScript code"
      id:
        type: string
      language:
        type: string
        enum: [typescript, javascript]
        default: typescript
      output:
        $ref: "#/$defs/CodeOutput"

  CodeOutput:
    type: object
    properties:
      stdout:
        type: string
      stderr:
        type: string
      result:
        description: "Return value"
      executionCount:
        type: integer

  PlotCell:
    type: object
    required: [plot]
    properties:
      plot:
        $ref: "#/$defs/PlotSpec"
      id:
        type: string
      output:
        $ref: "#/$defs/PlotOutput"

  PlotSpec:
    type: object
    required: [type]
    properties:
      type:
        type: string
        enum: [line, scatter, bar, histogram, contour, surface, parametric]
      data:
        type: array
        items:
          $ref: "#/$defs/PlotSeries"
      xaxis:
        $ref: "#/$defs/AxisSpec"
      yaxis:
        $ref: "#/$defs/AxisSpec"
      title:
        type: string
      width:
        type: integer
      height:
        type: integer

  PlotSeries:
    type: object
    properties:
      expression:
        type: string
      variable:
        type: string
      range:
        type: array
        items:
          type: number
        minItems: 2
        maxItems: 3
      label:
        type: string
      color:
        type: string

  AxisSpec:
    type: object
    properties:
      label:
        type: string
      range:
        type: array
        items:
          type: number
      scale:
        type: string
        enum: [linear, log, symlog]

  PlotOutput:
    type: object
    properties:
      svg:
        type: string
      png:
        type: string
        contentEncoding: base64

  TableCell:
    type: object
    required: [table]
    properties:
      table:
        $ref: "#/$defs/TableSpec"
      id:
        type: string

  TableSpec:
    type: object
    properties:
      headers:
        type: array
        items:
          type: string
      rows:
        type: array
        items:
          type: array
      expression:
        type: string
        description: "MathTS expression returning matrix/array"
      format:
        type: object
        additionalProperties:
          type: string
```

### Example Workbook

```yaml
# example.mtsw - MathTS Scientific Workbook
# A demonstration of the workbook format

version: "1.0"

metadata:
  title: "Projectile Motion Analysis"
  authors:
    - name: "Dr. Jane Smith"
      email: "jane.smith@university.edu"
      affiliation: "Department of Physics"
  created: "2024-12-05T10:00:00Z"
  description: |
    Analysis of projectile motion with air resistance.
    Demonstrates symbolic computation, unit handling, and plotting.
  tags: [physics, mechanics, differential-equations]
  kernel:
    name: mathts
    version: "1.0"
  settings:
    precision: 15
    unitSystem: SI
    outputFormat: latex

cells:
  # ═══════════════════════════════════════════════════════════════
  # INTRODUCTION
  # ═══════════════════════════════════════════════════════════════

  - markdown: |
      # Projectile Motion with Air Resistance

      ## Problem Statement

      A projectile is launched with initial velocity $v_0$ at angle $\theta$
      from the horizontal. We analyze its trajectory considering:

      1. Ideal case (no air resistance)
      2. Linear drag model ($F_d = -bv$)
      3. Quadratic drag model ($F_d = -cv^2$)

  # ═══════════════════════════════════════════════════════════════
  # PARAMETERS
  # ═══════════════════════════════════════════════════════════════

  - markdown: |
      ## Initial Conditions

  - math: "v0 = 50 m/s"
    label: v0
    id: "param-v0"
    output:
      value: { magnitude: 50, unit: "m/s" }
      latex: "v_0 = 50\\,\\mathrm{m/s}"

  - math: "theta = 45 deg"
    label: theta
    id: "param-theta"
    output:
      value: { magnitude: 0.7853981633974483, unit: "rad" }
      latex: "\\theta = 45°"

  - math: "g = 9.81 m/s^2"
    label: g
    id: "param-g"
    output:
      value: { magnitude: 9.81, unit: "m/s^2" }
      latex: "g = 9.81\\,\\mathrm{m/s^2}"

  # ═══════════════════════════════════════════════════════════════
  # IDEAL TRAJECTORY (No Air Resistance)
  # ═══════════════════════════════════════════════════════════════

  - markdown: |
      ## Ideal Trajectory

      Without air resistance, the equations of motion are:

  - math: |
      x(t) = v0 * cos(theta) * t
    label: x_ideal
    id: "eq-x-ideal"
    output:
      latex: "x(t) = v_0 \\cos(\\theta) \\cdot t"

  - math: |
      y(t) = v0 * sin(theta) * t - (1/2) * g * t^2
    label: y_ideal
    id: "eq-y-ideal"
    output:
      latex: "y(t) = v_0 \\sin(\\theta) \\cdot t - \\frac{1}{2} g t^2"

  - markdown: |
      ### Key Results

  - math: "t_flight = 2 * v0 * sin(theta) / g"
    label: t_flight
    id: "calc-tflight"
    output:
      value: { magnitude: 7.2136, unit: "s" }
      latex: "t_{\\text{flight}} = \\frac{2 v_0 \\sin\\theta}{g} = 7.21\\,\\mathrm{s}"

  - math: "range = v0^2 * sin(2*theta) / g"
    label: range
    id: "calc-range"
    output:
      value: { magnitude: 254.84, unit: "m" }
      latex: "R = \\frac{v_0^2 \\sin(2\\theta)}{g} = 254.84\\,\\mathrm{m}"

  - math: "max_height = v0^2 * sin(theta)^2 / (2*g)"
    label: max_height
    id: "calc-maxheight"
    output:
      value: { magnitude: 63.71, unit: "m" }
      latex: "h_{\\max} = \\frac{v_0^2 \\sin^2\\theta}{2g} = 63.71\\,\\mathrm{m}"

  # ═══════════════════════════════════════════════════════════════
  # TRAJECTORY PLOT
  # ═══════════════════════════════════════════════════════════════

  - markdown: |
      ### Trajectory Visualization

  - plot:
      type: parametric
      data:
        - expression: "[v0 * cos(theta) * t, v0 * sin(theta) * t - 0.5 * g * t^2]"
          variable: t
          range: [0, "t_flight", 0.01]
          label: "Ideal trajectory"
          color: "#2563eb"
      xaxis:
        label: "Horizontal distance (m)"
        range: [0, 280]
      yaxis:
        label: "Height (m)"
        range: [0, 80]
      title: "Projectile Trajectory"
      width: 800
      height: 500
    id: "plot-trajectory"
    output:
      svg: |
        <svg><!-- Generated SVG --></svg>

  # ═══════════════════════════════════════════════════════════════
  # NUMERICAL COMPARISON
  # ═══════════════════════════════════════════════════════════════

  - markdown: |
      ## Results Summary

  - table:
      headers: ["Model", "Flight Time (s)", "Range (m)", "Max Height (m)"]
      rows:
        - ["Ideal", "7.21", "254.84", "63.71"]
        - ["Linear Drag", "6.89", "231.52", "58.43"]
        - ["Quadratic Drag", "6.45", "198.76", "52.18"]
    id: "table-comparison"

  # ═══════════════════════════════════════════════════════════════
  # CUSTOM CODE
  # ═══════════════════════════════════════════════════════════════

  - markdown: |
      ## Custom Analysis

      We can also write TypeScript code for more complex calculations:

  - code: |
      // Numerical integration using RK4
      function rk4Trajectory(
        v0: number,
        theta: number,
        dragCoeff: number,
        dt: number = 0.01
      ): Array<[number, number]> {
        const trajectory: Array<[number, number]> = [];
        let x = 0, y = 0;
        let vx = v0 * Math.cos(theta);
        let vy = v0 * Math.sin(theta);

        while (y >= 0) {
          trajectory.push([x, y]);
          const v = Math.sqrt(vx*vx + vy*vy);
          const ax = -dragCoeff * v * vx;
          const ay = -9.81 - dragCoeff * v * vy;

          vx += ax * dt;
          vy += ay * dt;
          x += vx * dt;
          y += vy * dt;
        }

        return trajectory;
      }

      // Calculate trajectory with drag coefficient 0.1
      const dragTrajectory = rk4Trajectory(50, Math.PI/4, 0.1);
      console.log(`Final position: (${dragTrajectory.at(-1)?.[0].toFixed(2)}m, 0m)`);
    language: typescript
    id: "code-rk4"
    output:
      stdout: "Final position: (198.76m, 0m)"
      executionCount: 1

# ═══════════════════════════════════════════════════════════════
# STORED DATA (Computed Outputs)
# ═══════════════════════════════════════════════════════════════

data:
  executionState:
    scope:
      v0: { magnitude: 50, unit: "m/s" }
      theta: { magnitude: 0.7853981633974483, unit: "rad" }
      g: { magnitude: 9.81, unit: "m/s^2" }
      t_flight: { magnitude: 7.2136, unit: "s" }
      range: { magnitude: 254.84, unit: "m" }
      max_height: { magnitude: 63.71, unit: "m" }
    lastExecution: "2024-12-05T10:30:00Z"
```

### Comparison with Jupyter Format

| Aspect | Jupyter (.ipynb) | MathTS Workbook (.mtsw) |
|--------|------------------|-------------------------|
| Format | JSON | YAML |
| Readability | Poor (escaped strings) | Excellent (literal blocks) |
| Diff-ability | Hard (long lines) | Easy (natural line breaks) |
| Cell types | code, markdown, raw | math, code, markdown, plot, table |
| Math support | Via kernel | Native MathTS expressions |
| Units | None | First-class support |
| Outputs | Inline, verbose | Separate data section, compact |
| Schema | Informal | JSON Schema validated |

---

## 7. Core Components

### 7.1 Parser Module

```typescript
// @mathts/workbook/src/parser.ts

import * as yaml from 'yaml';
import Ajv from 'ajv';
import { Workbook, Cell, ParseOptions, ParseResult } from './types';
import workbookSchema from './schema/workbook.schema.json';

export class WorkbookParser {
  private ajv: Ajv;
  private validate: ValidateFunction;

  constructor() {
    this.ajv = new Ajv({ allErrors: true, strict: false });
    this.validate = this.ajv.compile(workbookSchema);
  }

  /**
   * Parse YAML string into Workbook object
   */
  parse(content: string, options?: ParseOptions): ParseResult<Workbook> {
    try {
      const doc = yaml.parse(content, {
        strict: false,
        prettyErrors: true,
      });

      if (!this.validate(doc)) {
        return {
          success: false,
          errors: this.validate.errors?.map(e => ({
            path: e.instancePath,
            message: e.message || 'Validation error',
            keyword: e.keyword,
          })),
        };
      }

      const workbook = this.transformToWorkbook(doc);
      return { success: true, data: workbook };
    } catch (error) {
      return {
        success: false,
        errors: [{ path: '', message: String(error) }],
      };
    }
  }

  /**
   * Parse from file path
   */
  async parseFile(filePath: string, options?: ParseOptions): Promise<ParseResult<Workbook>> {
    const content = await fs.readFile(filePath, 'utf-8');
    return this.parse(content, options);
  }

  private transformToWorkbook(doc: unknown): Workbook {
    // Transform raw YAML document to typed Workbook
  }
}
```

### 7.2 Executor Module

```typescript
// @mathts/workbook/src/executor.ts

import { MathTS } from '@mathts/core';
import {
  Workbook,
  Cell,
  ExecutionResult,
  ExecutionOptions,
  ExecutionContext
} from './types';

export class WorkbookExecutor {
  private math: MathTS;
  private context: ExecutionContext;

  constructor(options?: ExecutionOptions) {
    this.math = new MathTS(options?.mathts);
    this.context = this.createContext();
  }

  /**
   * Execute entire workbook
   */
  async execute(workbook: Workbook): Promise<ExecutionResult> {
    const results: CellResult[] = [];

    for (const cell of workbook.cells) {
      const result = await this.executeCell(cell);
      results.push(result);

      if (result.error && !this.context.options.continueOnError) {
        break;
      }
    }

    return {
      success: results.every(r => !r.error),
      cells: results,
      scope: this.math.exportScope(),
      executionTime: this.context.totalTime,
    };
  }

  /**
   * Execute single cell
   */
  async executeCell(cell: Cell): Promise<CellResult> {
    const startTime = performance.now();

    try {
      switch (cell.type) {
        case 'math':
          return this.executeMathCell(cell);
        case 'code':
          return this.executeCodeCell(cell);
        case 'plot':
          return this.executePlotCell(cell);
        case 'markdown':
          return { type: 'markdown', rendered: this.renderMarkdown(cell) };
        default:
          return { type: 'unknown', skipped: true };
      }
    } catch (error) {
      return {
        type: cell.type,
        error: String(error),
        executionTime: performance.now() - startTime,
      };
    }
  }

  private executeMathCell(cell: MathCell): CellResult {
    const result = this.math.evaluate(cell.math);

    if (cell.label) {
      this.math.set(cell.label, result.value);
    }

    return {
      type: 'math',
      value: result.value,
      latex: result.latex,
      display: result.display,
    };
  }

  private async executeCodeCell(cell: CodeCell): Promise<CellResult> {
    // Sandboxed TypeScript execution
    const sandbox = this.createSandbox();
    const transpiled = await this.transpileTypeScript(cell.code);

    const fn = new Function('math', 'console', 'scope', transpiled);
    const output: string[] = [];
    const mockConsole = {
      log: (...args: unknown[]) => output.push(args.map(String).join(' ')),
      // ... other console methods
    };

    const returnValue = fn(this.math, mockConsole, this.math.exportScope());

    return {
      type: 'code',
      stdout: output.join('\n'),
      result: returnValue,
    };
  }

  private executePlotCell(cell: PlotCell): CellResult {
    // Generate SVG plot from specification
    const svg = this.generatePlot(cell.plot);
    return { type: 'plot', svg };
  }

  /**
   * Reset execution state
   */
  reset(): void {
    this.math.clearScope();
    this.context = this.createContext();
  }
}
```

### 7.3 Serializer Module

```typescript
// @mathts/workbook/src/serializer.ts

import * as yaml from 'yaml';
import { Workbook, Cell, SerializeOptions } from './types';

export class WorkbookSerializer {
  /**
   * Serialize Workbook to YAML string
   */
  serialize(workbook: Workbook, options?: SerializeOptions): string {
    const doc = this.prepareDocument(workbook, options);

    return yaml.stringify(doc, {
      indent: 2,
      lineWidth: 0,          // No line wrapping
      defaultStringType: 'BLOCK_LITERAL',
      defaultKeyType: 'PLAIN',
      nullStr: '',
    });
  }

  /**
   * Serialize to file
   */
  async serializeToFile(
    workbook: Workbook,
    filePath: string,
    options?: SerializeOptions
  ): Promise<void> {
    const content = this.serialize(workbook, options);
    await fs.writeFile(filePath, content, 'utf-8');
  }

  /**
   * Export to Jupyter notebook format
   */
  toJupyterNotebook(workbook: Workbook): JupyterNotebook {
    return {
      nbformat: 4,
      nbformat_minor: 5,
      metadata: this.convertMetadata(workbook.metadata),
      cells: workbook.cells.map(cell => this.convertCell(cell)),
    };
  }

  /**
   * Import from Jupyter notebook
   */
  fromJupyterNotebook(notebook: JupyterNotebook): Workbook {
    return {
      version: '1.0',
      metadata: this.importMetadata(notebook.metadata),
      cells: notebook.cells.map(cell => this.importCell(cell)),
      data: this.importOutputs(notebook.cells),
    };
  }

  private prepareDocument(workbook: Workbook, options?: SerializeOptions): unknown {
    const doc: Record<string, unknown> = {
      version: workbook.version,
    };

    if (workbook.metadata) {
      doc.metadata = workbook.metadata;
    }

    doc.cells = workbook.cells.map(cell =>
      this.prepareCellForYaml(cell, options)
    );

    if (!options?.excludeOutputs && workbook.data) {
      doc.data = workbook.data;
    }

    return doc;
  }
}
```

### 7.4 Jupyter Interoperability

```typescript
// @mathts/jupyter/src/notebook.ts

/**
 * Jupyter notebook format types (nbformat v4)
 */
export interface JupyterNotebook {
  nbformat: 4;
  nbformat_minor: number;
  metadata: NotebookMetadata;
  cells: JupyterCell[];
}

export interface NotebookMetadata {
  kernelspec?: KernelSpec;
  language_info?: LanguageInfo;
  [key: string]: unknown;
}

export type JupyterCell = CodeCell | MarkdownCell | RawCell;

export interface CodeCell {
  cell_type: 'code';
  id?: string;
  source: string | string[];
  metadata: CellMetadata;
  execution_count: number | null;
  outputs: Output[];
}

export interface MarkdownCell {
  cell_type: 'markdown';
  id?: string;
  source: string | string[];
  metadata: CellMetadata;
}

// ... additional types
```

```typescript
// @mathts/jupyter/src/import.ts

import { Workbook, Cell } from '@mathts/workbook';
import { JupyterNotebook, JupyterCell } from './notebook';

export function importNotebook(notebook: JupyterNotebook): Workbook {
  const cells: Cell[] = [];
  const outputs: Map<string, unknown> = new Map();

  for (const jupyterCell of notebook.cells) {
    const { cell, output } = convertJupyterCell(jupyterCell);
    cells.push(cell);

    if (output) {
      outputs.set(cell.id!, output);
    }
  }

  return {
    version: '1.0',
    metadata: convertNotebookMetadata(notebook.metadata),
    cells,
    data: outputs.size > 0 ? { outputs: Object.fromEntries(outputs) } : undefined,
  };
}

function convertJupyterCell(cell: JupyterCell): { cell: Cell; output?: unknown } {
  const source = Array.isArray(cell.source)
    ? cell.source.join('')
    : cell.source;

  switch (cell.cell_type) {
    case 'markdown':
      return {
        cell: {
          type: 'markdown',
          id: cell.id || generateId(),
          markdown: source,
        },
      };

    case 'code':
      // Detect if this is a math expression or general code
      if (isMathExpression(source)) {
        return {
          cell: {
            type: 'math',
            id: cell.id || generateId(),
            math: source,
          },
          output: cell.outputs.length > 0 ? convertOutputs(cell.outputs) : undefined,
        };
      }

      return {
        cell: {
          type: 'code',
          id: cell.id || generateId(),
          code: source,
          language: detectLanguage(notebook.metadata),
        },
        output: convertCodeOutputs(cell.outputs),
      };

    default:
      return {
        cell: {
          type: 'markdown',
          id: generateId(),
          markdown: source,
        },
      };
  }
}
```

---

## 8. API Design

### Public API Surface

```typescript
// @mathts/core - Mathematical Engine
export { MathTS } from './engine';
export type {
  MathResult,
  SymbolicExpression,
  TypedMatrix,
  Quantity,
  MathTSConfig,
} from './types';

// @mathts/workbook - Workbook Format
export { WorkbookParser } from './parser';
export { WorkbookSerializer } from './serializer';
export { WorkbookExecutor } from './executor';
export type {
  Workbook,
  Cell,
  MathCell,
  CodeCell,
  MarkdownCell,
  PlotCell,
  TableCell,
  ExecutionResult,
} from './types';

// @mathts/cli - Command Line Interface
export { run } from './cli';

// @mathts/jupyter - Jupyter Interop
export { importNotebook, exportNotebook } from './convert';
export type { JupyterNotebook, JupyterCell } from './notebook';
```

### CLI Commands

```bash
# Convert between formats
mathts convert input.ipynb output.mtsw
mathts convert input.mtsw output.ipynb
mathts convert input.mtsw output.md    # Markdown export
mathts convert input.mtsw output.tex   # LaTeX export

# Execute workbook
mathts run workbook.mtsw                     # Execute and update outputs
mathts run workbook.mtsw --output result.mtsw
mathts run workbook.mtsw --format json       # JSON output to stdout
mathts run workbook.mtsw --cell 5            # Run up to cell 5

# Validate workbook
mathts validate workbook.mtsw
mathts validate --schema custom-schema.json workbook.mtsw

# Initialize new workbook
mathts init my-project.mtsw
mathts init --template physics my-project.mtsw

# Watch mode (re-execute on changes)
mathts watch workbook.mtsw

# REPL mode
mathts repl
mathts repl --load workbook.mtsw
```

### Programmatic API Examples

```typescript
import { MathTS } from '@mathts/core';
import { WorkbookParser, WorkbookExecutor, WorkbookSerializer } from '@mathts/workbook';
import { importNotebook } from '@mathts/jupyter';

// === Basic MathTS Usage ===
const math = new MathTS();
const result = math.evaluate('integrate(x^2, x)');
console.log(result.latex); // \frac{x^3}{3}

// === Parse and Execute Workbook ===
const parser = new WorkbookParser();
const executor = new WorkbookExecutor();
const serializer = new WorkbookSerializer();

// Parse YAML workbook
const parseResult = await parser.parseFile('analysis.mtsw');
if (!parseResult.success) {
  console.error('Parse errors:', parseResult.errors);
  process.exit(1);
}

// Execute all cells
const workbook = parseResult.data;
const execResult = await executor.execute(workbook);

// Update workbook with outputs
workbook.cells.forEach((cell, i) => {
  if (execResult.cells[i].output) {
    cell.output = execResult.cells[i].output;
  }
});

// Save with outputs
await serializer.serializeToFile(workbook, 'analysis-executed.mtsw');

// === Jupyter Interoperability ===
import * as fs from 'fs';

const notebookJson = JSON.parse(fs.readFileSync('legacy.ipynb', 'utf-8'));
const converted = importNotebook(notebookJson);
await serializer.serializeToFile(converted, 'converted.mtsw');

// === Streaming Execution ===
for await (const cellResult of executor.executeStream(workbook)) {
  console.log(`Cell ${cellResult.index}: ${cellResult.status}`);
  if (cellResult.output) {
    console.log(cellResult.output.display);
  }
}
```

---

## 9. Feature Roadmap

### Phase 1: Foundation (MVP)

**Goal:** TypeScript port with basic functionality

| Feature | Priority | Status |
|---------|----------|--------|
| TypeScript project setup (monorepo) | P0 | Planned |
| Core type definitions | P0 | Planned |
| YAML parser with schema validation | P0 | Planned |
| Basic YAML ↔ ipynb conversion | P0 | Planned |
| CLI: convert command | P0 | Planned |
| Unit tests (Jest) | P0 | Planned |

**Deliverables:**
- `@mathts/workbook` package
- `@mathts/jupyter` package
- `@mathts/cli` package (convert only)

### Phase 2: MathTS Engine

**Goal:** Math.js integration with TypeScript wrapper

| Feature | Priority | Status |
|---------|----------|--------|
| MathTS engine wrapper | P0 | Planned |
| Expression evaluation | P0 | Planned |
| Symbolic operations (simplify, expand) | P1 | Planned |
| Differentiation & integration | P1 | Planned |
| Equation solving | P1 | Planned |
| Unit system | P1 | Planned |
| Matrix operations | P1 | Planned |
| Result serialization (YAML-friendly) | P0 | Planned |

**Deliverables:**
- `@mathts/core` package

### Phase 3: Workbook Execution

**Goal:** Execute workbooks headlessly

| Feature | Priority | Status |
|---------|----------|--------|
| Cell execution engine | P0 | Planned |
| Scope management | P0 | Planned |
| Math cell execution | P0 | Planned |
| Code cell execution (sandboxed) | P1 | Planned |
| Execution state persistence | P1 | Planned |
| CLI: run command | P0 | Planned |
| CLI: watch mode | P2 | Planned |

### Phase 4: Visualization

**Goal:** Plot generation and table formatting

| Feature | Priority | Status |
|---------|----------|--------|
| Plot cell support | P1 | Planned |
| SVG generation (D3.js or similar) | P1 | Planned |
| Table cell rendering | P1 | Planned |
| LaTeX output generation | P2 | Planned |

### Phase 5: Advanced Features

**Goal:** Production-ready scientific workbench

| Feature | Priority | Status |
|---------|----------|--------|
| REPL mode | P2 | Planned |
| Export to Markdown | P2 | Planned |
| Export to LaTeX/PDF | P2 | Planned |
| Custom function definitions | P2 | Planned |
| Workbook templates | P3 | Planned |
| VS Code extension | P3 | Planned |
| Web-based viewer | P3 | Planned |

### Phase 6: Ecosystem

**Goal:** Integrations and community features

| Feature | Priority | Status |
|---------|----------|--------|
| JupyterLab extension | P3 | Planned |
| Python interop (via WASM) | P3 | Planned |
| Package registry for functions | P3 | Planned |
| Collaborative editing foundation | P4 | Future |

---

## 10. Technical Requirements

### Runtime Requirements

| Requirement | Specification |
|-------------|---------------|
| Node.js | >= 18.0.0 (LTS) |
| TypeScript | >= 5.0 |
| ES Target | ES2022 |
| Module System | ESM (with CJS compatibility layer) |

### Dependencies

**Core Dependencies:**

```json
{
  "dependencies": {
    "mathjs": "^12.0.0",
    "yaml": "^2.3.0",
    "ajv": "^8.12.0",
    "ajv-formats": "^2.1.1"
  }
}
```

**CLI Dependencies:**

```json
{
  "dependencies": {
    "commander": "^11.0.0",
    "chalk": "^5.3.0",
    "ora": "^7.0.0"
  }
}
```

**Development Dependencies:**

```json
{
  "devDependencies": {
    "typescript": "^5.3.0",
    "vitest": "^1.0.0",
    "@types/node": "^20.0.0",
    "tsx": "^4.0.0",
    "tsup": "^8.0.0",
    "turbo": "^1.11.0",
    "prettier": "^3.1.0",
    "eslint": "^8.55.0",
    "@typescript-eslint/eslint-plugin": "^6.0.0"
  }
}
```

### Build Configuration

```json
// tsconfig.base.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": false
  }
}
```

### Quality Standards

| Metric | Target |
|--------|--------|
| Test Coverage | >= 80% |
| Type Coverage | 100% (strict mode) |
| Documentation | JSDoc for all public APIs |
| Bundle Size | < 500KB (core), < 1MB (full) |
| Lighthouse Score | N/A (library) |

### Security Considerations

1. **Code Execution Sandboxing**
   - Code cells run in isolated context
   - No file system access by default
   - Network access disabled
   - Configurable permissions model

2. **YAML Parsing**
   - Use safe parsing (no code execution in YAML)
   - Schema validation before processing
   - Input size limits

3. **Dependency Security**
   - Regular dependency audits
   - Minimal dependency footprint
   - No native modules (pure JS/TS)

---

## 11. Migration Strategy

### From ipyaml (Python)

The TypeScript implementation maintains **API compatibility** where applicable:

| Python (ipyaml) | TypeScript (MathTS) |
|-----------------|---------------------|
| `nb_to_yaml(nb)` | `serializer.serialize(workbook)` |
| `yaml_to_nb(data)` | `parser.parse(yaml)` |
| `ipyaml input.ipynb output.yaml` | `mathts convert input.ipynb output.mtsw` |
| `YAMLContentsManager` | Jupyter extension (future) |

### Migration Path for Users

1. **Existing YAML notebooks** (from ipyaml):
   - Automatically importable with version detection
   - One-time migration to new schema

2. **Jupyter notebooks** (.ipynb):
   - Full bidirectional conversion
   - Math cells auto-detected from code patterns

3. **Maple worksheets** (.mw, .mws):
   - Future: Import tool for basic worksheets
   - Manual conversion guide provided

### Backward Compatibility

```typescript
// Legacy ipyaml format detection
export function detectFormat(content: string): 'ipyaml-v1' | 'mathts-v1' {
  const doc = yaml.parse(content);

  if (doc.version?.startsWith('1.')) {
    return 'mathts-v1';
  }

  // Legacy format: has 'cells' array but no 'version'
  if (Array.isArray(doc.cells) && !doc.version) {
    return 'ipyaml-v1';
  }

  throw new Error('Unknown workbook format');
}

// Automatic migration
export function migrateToV1(legacy: LegacyWorkbook): Workbook {
  return {
    version: '1.0',
    metadata: legacy.metadata || {},
    cells: legacy.cells.map(migrateCell),
    data: legacy.data,
  };
}
```

---

## Appendices

### A. Glossary

| Term | Definition |
|------|------------|
| **Cell** | Atomic unit of content in a workbook (math, code, markdown, etc.) |
| **MathTS** | TypeScript wrapper around Math.js providing type-safe math operations |
| **Scope** | Variable namespace persisted across cell executions |
| **Workbook** | Complete document containing cells, metadata, and computed outputs |
| **YAML Block Scalar** | YAML syntax (`|`) preserving literal multi-line strings |

### B. File Format Examples

See `/examples/` directory in repository for:
- `basic.mtsw` - Minimal workbook
- `physics.mtsw` - Physics calculations with units
- `linear-algebra.mtsw` - Matrix operations
- `calculus.mtsw` - Symbolic differentiation/integration
- `data-analysis.mtsw` - Statistical analysis

### C. References

- [Math.js Documentation](https://mathjs.org/docs/)
- [Jupyter Notebook Format (nbformat)](https://nbformat.readthedocs.io/)
- [YAML 1.2 Specification](https://yaml.org/spec/1.2.2/)
- [JSON Schema](https://json-schema.org/)
- [Maple Worksheet Reference](https://www.maplesoft.com/support/help/)

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0-draft | 2024-12-05 | MathTS Team | Initial specification |

---

*This specification is a living document and will be updated as the project evolves.*
