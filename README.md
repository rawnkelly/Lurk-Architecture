Lurk Terminal — Technical Architecture
Overview

Lurk Terminal is a real-time cross-platform prediction market terminal for Polymarket and Kalshi. The MVP is a single-screen execution environment built around four features:

Arb Scanner

Threshold Alerts

Signal Search

Performance Tracker

The core differentiator is honest net spread after fees, not gross spread. The scanner, divergence panel, and fee drawer should all use one shared fee calculator as source of truth. Whale watching, leaderboards, public profiles, API access, paper trading, AI sentiment, and community extras are explicitly out of MVP.


Core Features
1. Arb Scanner

Real-time fee-adjusted net spread detection between Polymarket and Kalshi.

How it works:

backend ingests and normalizes both markets

mapping engine pairs equivalent contracts

spread engine computes gross and net

market rows are written to market_data

frontend refreshes every 3 seconds

scanner is sorted by net spread descending

Scanner columns:

Market name

Polymarket price

Kalshi price

Net spread

Time detected

Color logic:

green if above threshold

grey if below threshold

never red

Starter users see only the first few rows clearly; rows below are blurred with upgrade treatment.

2. Threshold Alerts

Single-number alert threshold, defaulting to 5%.

Behavior:

no save button

updates on change

email alerts for Starter

mobile push for paid tiers

alert deep-links directly into the relevant market row pre-expanded

This lives in the bottom section of the right panel, not on a separate page.

3. Signal Search

Claude-powered web search summarized by topic / keyword.

Behavior:

opened from the left panel

panel shows saved searches and inline results

platform selector changes prompt bias, not infrastructure

usage tracked monthly by plan

Starter gets limited free usage, paid tiers get more capacity

This is the research layer, not the main execution surface.

4. Performance Tracker

Automatic trade logging and performance history.

Behavior:

collapsed bottom cockpit is always visible

expands into a fuller trade history view

no manual journaling required

tracks total P&L, win rate, average net spread captured, and trade count

“Log this trade” is triggered from divergence panel / fee drawer

This is part of the same terminal screen, not a separate product area.

Screen Layout
Left Panel — Signal Search

hidden by default on desktop

opened from TopBar

overlays on mobile

contains saved searches, create-new button, inline results

replaces old signal feed concept

Center — Scanner Table

owns the main center space

no chart in MVP

clicking a row opens the fee breakdown drawer

refreshes every 3 seconds

sorted by net spread descending

Right Panel — Divergence + Threshold

always visible on desktop

slide-out on mobile

top section shows selected market:

Polymarket price

Kalshi price

gross spread

fee breakdown

net spread

log trade CTA

bottom section contains threshold input

Bottom Cockpit — Performance

collapsed summary always visible

expands to trade history and stat cards

Starter banner can live inline here later

That is the real terminal composition. No second tab belongs in this doc.

Database Schema
Existing tables already in the product

users

subscriptions

lurks

lurk_results

user_preferences

Terminal / scanner tables

The live pipeline status doc reflects the current backend reality:

market_mappings

active source of truth for monitored pairs

market_data

live spread rows written by main.py

candidate_matches

AI matching pipeline results

raw_markets

stored active Polymarket markets

The current pipeline has 77 active mappings loaded into the live loop, and market_data is now receiving filtered live output after the Kalshi normalization fix.

You should also keep / add application-layer tables for:

trade logs

signal search history

lurk usage counts

Those align with the MVP infrastructure list.

Technical Stack
Frontend

Next.js 15

React 19

Tailwind CSS + shadcn/ui

TanStack Table for scanner

existing auth/billing UI stack

Backend

Python data pipeline for ingestion, mapping, spread computation

Next.js API routes for app-facing endpoints

PostgreSQL / Supabase

Redis for caching and throttling

Claude-powered Signal Search

Data Sources

Polymarket

Kalshi API

Claude web search for Signal Search

Infrastructure

Vercel for frontend

long-running Python worker for live pipeline

Supabase for DB

Clerk for auth

Stripe for billing

Resend for email alerts

This is the actual MVP-shaped stack, not the older whale/copy-trading/leaderboard architecture.

What’s Already Working

From the current pipeline status:

Polymarket ingestion

Kalshi ingestion

market mapping engine

live pass across 77 active mappings

quality filtering

DB upsert into market_data

first successful quality-passing spread detection

That means the backend is now alive enough to support terminal integration, though it still needs fee-adjusted net spread as the frontend-facing truth.

Critical Remaining Gap

The main remaining product gap is still:

gross spread vs. net spread

The MVP spec requires:

net spread in the scanner

fee breakdown in the divergence panel

fee breakdown in the drawer

one shared fee-calculator.ts utility used everywhere

Until that is wired through, the backend is useful but not yet product-correct.

Explicitly Out of MVP

Remove these from the architecture doc entirely:

leaderboards

public profiles

whale watching

copy trading

raw API access

paper trading

AI sentiment

community beyond events

Lurk AI as an active MVP feature

Those are explicitly out of MVP.
