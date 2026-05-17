# PakWheels Website — Technical Documentation

> Full-stack design and C++ class mapping reference for all screens, entities, and relationships.

---

## Table of Contents

1. [Home / Listings Page](#1-home--listings-page)
2. [Listing Detail Page](#2-listing-detail-page)
3. [Search Results Page](#3-search-results-page)
4. [Buyer Dashboard](#4-buyer-dashboard-ali-khan)
5. [Seller Dashboard](#5-seller-dashboard-rashid-motors)
6. [Admin Panel](#6-admin-panel)
7. [Post New Listing](#7-post-new-listing)
8. [UML Class Diagram](#8-uml-class-diagram)

---

## 1. Home / Listings Page

> Main landing page with search, filters, listing cards, and the messaging panel.

This screen maps directly to the `Marketplace` class methods: `searchListings()`, `filterByMaxPrice()`, and the `User` messaging system.

![Home / Listings Page](./images/screen1_home.png)
*Figure 1 — Home page: search, filters, listing cards (`Listing` objects), and messaging panel*

### C++ Mappings

| UI Element | C++ Mapping |
|---|---|
| Listing cards | `Listing` class with `Vehicle*` (composition) + `Location` (embedded) |
| Search bar | `Marketplace::searchListings()` delegating to `ISearchable::matchesFilter()` |
| Filter chips | `Marketplace::filterByMaxPrice()` and `filterByYearRange()` |
| Approved badge | `IApprovable::approve()` called by `Admin` via `Marketplace` |
| Stats bar | `User::getTotalUsers()`, `Vehicle::getTotalVehicles()`, `Listing::getPlatformFee()` |
| Message panel | `User::receiveMessage()`, `IMessagable::sendMessage()`, `Message` objects |

---

## 2. Listing Detail Page

> Individual listing page for **L001: 2022 Honda Civic**.

Shows all attributes of the `Car` subclass (`engineCC`, `transmission`, `bodyType`, `doors`, `isAC`), the Seller's verification status, `Location`, and messaging CTA.

![Listing Detail Page](screen2_listing_detail.png)
*Figure 2 — Listing detail: `Car::displayDetails()`, seller panel, location, and messaging CTA*

### C++ Mappings

| UI Element | C++ Mapping |
|---|---|
| Vehicle specs | `Car::displayDetails()` — `engineCC`, `transmission`, `bodyType`, `numDoors`, `isAC` |
| Seller panel | `Seller::isVerified`, `Seller::averageRating`, `Seller::shopName` |
| Location box | `Location::toString()` — `'Karachi, Sindh, Pakistan'` |
| Approved badge | `Listing::isApproved()` via `IApprovable` |
| Message CTA | `Marketplace::sendMessage(fromID, toID, content, timestamp)` |
| Favourites btn | `Buyer::saveFavourite(listingID)` |
| Annual Tax est | `Car::estimateAnnualTax()` — 1500cc → Rs 25,000 |

---

## 3. Search Results Page

> Results filtered by `brand = 'Honda'`.

Shows the output of `Marketplace::searchListings("brand", "Honda")` — only approved listings where `Vehicle::matchesFilter()` returns `true` for `brand = Honda`.

![Search Results Page](screen3_search_results.png)
*Figure 3 — Search results: polymorphic `Vehicle*` display, approved/pending guards, pagination*

### C++ Mappings

| UI Element | C++ Mapping |
|---|---|
| Active filters | `Marketplace::searchListings("brand","Honda")` + `searchListings("city","Karachi")` |
| Result count | Static counter: `Marketplace::totalSearches++` on each query |
| Listing cards | `Listing::isApproved()` guard before displaying results |
| Honda CD-70 | `Bike` subclass displayed via `Vehicle*` base pointer (polymorphism) |
| Pending badge | `Listing::approved == false` → `IApprovable::isApproved()` returns `false` |
| Pagination | Array iteration: `listings[0..listingCount]` in `Marketplace` |

---

## 4. Buyer Dashboard (Ali Khan)

> Ali Khan's (`B001`) personal dashboard.

Shows saved favourites (`L001`, `L005`), the `friend` function `canAfford()` budget check, recent messages sent to sellers, and Buyer-specific attributes: `maxBudget`, `preferredCity`.

![Buyer Dashboard](screen4_buyer_dashboard.png)
*Figure 4 — Buyer dashboard: favourites array, `canAfford()` friend function, message inbox*

### C++ Mappings

| UI Element | C++ Mapping |
|---|---|
| Favourites | `Buyer::saveFavourite("L001")`, `Buyer::viewFavourites()`, `favourites[20]` array |
| Budget check | `friend bool canAfford(const Buyer& buyer, double price)` — accesses private `maxBudget` |
| L001 (Rs 32L) | `canAfford(*ali, 3200000)` → **NO** (budget = Rs 25,00,000) |
| L004 (Rs 3.2L) | `canAfford(*ali, 320000)` → **YES** |
| Messages sent | `Marketplace::sendMessage("B001","S001", msg, timestamp)` |
| Stats widgets | `Buyer::favCount`, `Buyer::messageCount` (inbox array), static `totalSearches` |
| Remove fav | `Buyer::removeFavourite("L005")` — string array scan + shift |

---

## 5. Seller Dashboard (Rashid Motors)

> Rashid Motors (`S001`) seller view.

Shows all 3 approved listings (`L001`, `L002`, `L005`), unread messages from Ali Khan, rating averaged by `updateRating()`, and the Rs 500 platform fee notice mapping to `PLATFORM_FEE` static const.

![Seller Dashboard](screen5_seller_dashboard.png)
*Figure 5 — Seller dashboard: listings array, unread messages, rating, platform fee notice*

### C++ Mappings

| UI Element | C++ Mapping |
|---|---|
| My listings | `Listing* listings[]` filtered by `sellerID == "S001"` |
| Platform fee | `static const double Listing::PLATFORM_FEE = 500.0` |
| Unread messages | `Message::isRead == false` in `Seller::inbox[50]` |
| Rating display | `Seller::averageRating` updated via `updateRating(r.getRating())` |
| Verified badge | `Seller::isVerified` set by `Admin::promoteToVerified(*rashid)` |
| CNIC verified | `friend bool verifyCNIC(const Seller& s, const string& cnic)` |
| Edit / Delete | `Marketplace::updateListingPrice()`, `Marketplace::deleteListing()` |

---

## 6. Admin Panel

> Dark-themed admin interface for System Admin (`A001`).

Shows the approval queue, user management with ban/promote actions, `friend` function results for CNIC and admin code verification, and platform-wide statistics.

![Admin Panel](screen6_admin_panel.png)
*Figure 6 — Admin panel: approval queue, user management table, friend function results, platform stats*

### C++ Mappings

| UI Element | C++ Mapping |
|---|---|
| Approve listing | `Marketplace::approveListing("L001", *superAdmin)` → `IApprovable::approve()` |
| Reject listing | `market.rejectListing("L006", "Missing registration documents", *superAdmin)` |
| Ban user | `Admin::banUser(User& u)` — checks `canBanUsers` before calling `u.deactivate()` |
| Promote seller | `Admin::promoteToVerified(Seller& s)` → `s.verify()`, increments `casesHandled` |
| CNIC verify | `friend bool verifyCNIC(*rashid, "42101-1234567-1")` → **MATCH** |
| Admin code | `friend bool validateAdminCode(*superAdmin, "ADMIN-2026")` → **VALID** |
| Stats | `User::getTotalUsers() = 5`, `Vehicle::getTotalVehicles() = 6`, `totalSearches = 4` |

---

## 7. Post New Listing

> Form a Seller fills in to create a new `Listing`.

All form fields map directly to `Car` constructor parameters (`brand`, `model`, `year`, `mileage`, `fuelType`, `color`, `numDoors`, `transmission`, `bodyType`, `engineCC`, `isAC`) and `Listing` constructor parameters (`listingID`, `askingPrice`, `description`, `sellerID`, `vehicle`, `location`).

![Post New Listing](screen7_post_listing.png)
*Figure 7 — Post listing form: all fields map to `Car` and `Listing` constructor parameters*

### C++ Mappings

| Form Field | C++ Mapping |
|---|---|
| Vehicle Type | Determines subclass: `Car` / `Bike` / `Truck` (polymorphic instantiation) |
| Brand / Model | `Vehicle::brand`, `Vehicle::model` (base class private members) |
| Year / Mileage | `Vehicle::year` (`int`), `Vehicle::mileage` (`double`) |
| Engine CC | `Car::engineCC` (`double`) — used by `estimateAnnualTax()` |
| Transmission | `Car::transmission` (`string`) — `matchesFilter("transmission", val)` |
| City / Province | `Location(city, province)` — embedded by value in `Listing` |
| Asking Price | `Listing::askingPrice` — used by `operator<` and `filterByMaxPrice()` |
| Fee notice | `static const double Listing::PLATFORM_FEE = 500.0` |
| Submit button | `market.addListing(l)` → pending admin approval (`approved = false`) |

---

## 8. UML Class Diagram

> Full class hierarchy showing all **13 classes** and **4 interfaces**.

![UML Class Diagram](screen8_uml_diagram.png)
*Figure 8 — Complete UML: inheritance, interface implementation, composition, and aggregation*

### Relationship Types

| Symbol | Meaning |
|---|---|
| Solid arrow `→` | Inheritance |
| Dashed arrow `⇢` | Interface implementation |
| Filled diamond `◆` | Composition (owns / deletes) |
| Open diamond `◇` | Aggregation (references / borrows) |

### Class Overview

| Category | Members |
|---|---|
| **Interfaces** | `IDisplayable`, `ISearchable`, `IApprovable`, `IMessagable` |
| **Abstract bases** | `Vehicle` (`IDisplayable` + `ISearchable`), `User` (`IDisplayable` + `IMessagable`) |
| **Vehicle subclasses** | `Car`, `Bike`, `Truck` — override `getType()` and `displayDetails()` |
| **User subclasses** | `Buyer`, `Seller`, `Admin` — override `getRole()` and `displayDetails()` |
| **Listing** | Composition with `Vehicle*` (owned/deleted) and `Location` (by value) |
| **Marketplace** | Composition with `Listing*[]` (owns) · Aggregation with `User*[]` (borrows) |
| **Friend functions** | `canAfford(Buyer)`, `verifyCNIC(Seller)`, `validateAdminCode(Admin)` |
| **Operator overloads** | `Vehicle op>` · `Listing op< op+ op==` · `Location op==` · `Review op>` |

### Full Relationship Summary

```
IDisplayable  ◄── Vehicle ──► ISearchable
                    │
          ┌─────────┼─────────┐
         Car       Bike      Truck

IDisplayable  ◄── User ──► IMessagable
                   │
         ┌─────────┼─────────┐
       Buyer    Seller     Admin

Listing ◆────► Vehicle*   (composition — Listing owns Vehicle)
Listing ◆────► Location   (composition — embedded by value)

Marketplace ◆────► Listing*[]  (composition — Marketplace owns listings)
Marketplace ◇────► User*[]     (aggregation — Marketplace borrows users)

Review ◇────► Buyer / Seller   (aggregation — references by ID)
```

