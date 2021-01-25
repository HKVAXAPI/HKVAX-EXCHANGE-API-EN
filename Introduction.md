# Introduction

## Basic Introduction

Welcome to the HKVAX developer documentation. This document outlines the API introduction, access method and interface description of HKVAX.

API is divided into three categories: quotation, trade, and user information query. You can create API Keys with different permissions according to your needs, and use the APIs for automated transactions.

The quotation APIs provides market quotation data, and all quotation interfaces are public. The trading and user query APIs requires identity verification and provides functions such as placing and canceling orders, and querying order and account information.

## Trading Rules

### Matching Engine

HKVAX matches according to the rule of "price priority-user priority-time priority". The rules are the same for all order types.

Price priority means that a buy order with a higher price has priority over a buy order with a lower price, and a sell order with a lower price has priority over a sell order with a higher price.

User priority means that if the price of some pending orders is the same, users’ orders will enjoy a higher priority than orders placed by HKVAX’s company accounts or employees’ accounts.

Time priority means that for pending orders with the same price and the same user type, the order sequence is determined by the time of placing orders, the earlier placed orders are prioritized than the later placed orders.

### Order Lifecycle

After the order enters the matching engine, it is "NOT FILLED" `MISMATCH` status;

Then the order starts to be filled, and the partially filled order will become ``PARTIAL `` status;

Until an order is all filled, it will become "deal" `DEAL` status.

If an unfilled order is successfully canceled, then it becomes `CANCEL_MISMATCH` status;

If a partially filled order is successfully canceled, then it becomes `CANCEL_PARTIAL` status;

In addition, if the system is processing the order cancellation and there is no result response, then the order is "being cancelled" `FIRSTTRIAL ` status.

## Self-Trade Prevention 

Self-trading is not allowed on HKVAX. Two orders from the same user will not fill one another. When a user's order is matching, if the counterparty is himself, the system will automatically cancel the later order  to prevent self-trading.

## Fees

To enhance market liquidity and participation, HKVAX adopts a maker-taker fee schedule, in which the maker fee is lower than the taker fee. Maker: 0.2%, Maker: 0.1%

## Server

HKVAX's database and servers are based in Hong Kong.

## Access URLs

https://api.trade.hkvax.com/v1

## Mock Environment

The mock environment can be used to test API connectivity and trading. The function of the mock environment is independent of the formal environment. You need to register, log in, and create an API key separately.

In addition, once you complete the registration, we will provide sufficient simulated funds for you to test.

### Mock Environment URLs

When testing API connectivity, make sure to use the following URL:

**website**: [https://simtrade.hkvax.com](https://simtrade.hkvax.com/)

**REST API**: [https://api.simtrade.hkvax.com/v1 ](https://api.simtrade.hkvax.com/v1 )
