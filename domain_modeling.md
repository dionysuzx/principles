---
name: shared-language-domain-modeling
description: Use when asked to explore, design, or understand a product by writing a short F# shared-language domain model in the style of Scott Wlaschin: types first, workflows second, implementation later.
---

# Shared-Language Domain Modeling

Write a small F# model that shows the business language before the implementation.

## Do

1. Start from the shared language: nouns, states, roles, rules, and workflows.
2. Model nouns with records and choices with discriminated unions.
3. Model workflows as function types: `Input -> Output`.
4. Keep it short, usually under 100 lines.
5. Use this before writing code, while designing, or after reading code to clarify what already exists.

## Avoid

- Frameworks, storage, HTTP, CLI, UI, and database details.
- Generic buckets like `Data`, `Manager`, `Service`, `Utils`, or `Dto`.
- Implementations, algorithms, or smart constructors unless validation is the point.

## Shape

Use this kind of shape:

```fsharp
module CardGame

type Suit = Club | Diamond | Spade | Heart
type Rank = Two | Three | Four | Five | Six | Seven | Eight | Nine | Ten | Jack | Queen | King | Ace
type Card = Suit * Rank

type Hand = Card list
type Deck = Card list
type Player = { Name: string; Hand: Hand }
type Game = { Deck: Deck; Players: Player list }

type Deal = Deck -> Deck * Card
type PickupCard = Hand * Card -> Hand
```

The output should read like something a domain expert and programmer could point at together and say, "yes, that is the product."
