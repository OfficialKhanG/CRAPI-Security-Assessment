# 🔒 CRAPI Security Assessment

## Complete Vulnerability Analysis Report

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Security Research](https://img.shields.io/badge/Security-Research-red.svg)](https://github.com)

## 📋 Executive Summary

This repository contains a **comprehensive security assessment** of CRAPI (Comprehensive REST API). The assessment uncovered **4 critical vulnerabilities**.

## 🎯 Vulnerabilities Discovered

| # | Vulnerability | Severity |
|---|---------------|----------|
| 1 | Order Return BOLA | 🔴 High |
| 2 | Vehicle Location Tracking | 🔴 Critical |
| 3 | Mass Assignment - Admin Creation | 🔴 Critical |
| 4 | Negative Price Exploit | 🔴 Critical |

## 🚀 Quick Start

```bash
# Clone repository
git clone https://github.com/YOUR_USERNAME/CRAPI-Security-Assessment.git
cd CRAPI-Security-Assessment

# Make scripts executable
chmod +x exploits/*.sh

# Run exploits
./exploits/order_bola.sh
./exploits/admin_creation.sh
./exploits/negative_price.sh
./exploits/vehicle_tracking.sh
