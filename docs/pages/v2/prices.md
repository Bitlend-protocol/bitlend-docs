---
layout: docs-content
title: Compound II | Docs - Open Price Feed
permalink: /v2/prices/
docs_version: v2

## Element ID: In-page Heading
sidebar_nav_data:
  open-price-feed: Open Price Feed
  architecture: Architecture
  price: Get Price
  underlying-price: Get Underlying Price
---

# Open Price Feed

## Introduction

The Open Price Feed accounts price data for the Compound protocol. The protocol's Comptroller contract uses it as a source 
of truth for prices. Prices are updated by [Band Standard Dataset](https://docs.bandchain.org/band-standard-dataset/).


The Band Standard Dataset prices are used directly in the Comptroller through the price feed as there is no support for 
an anchored view. Below are some of the
