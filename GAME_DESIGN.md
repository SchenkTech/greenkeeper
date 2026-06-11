# GreenKeeper — Game Design Document

## Concept

A golf club management simulator. You inherit a struggling 9-hole club and must grow it into a renowned destination course. Place and upgrade holes, hire staff, set prices, run events, and watch your reputation — and revenues — climb.

Kairosoft-style: visual course map you actually build, layered over menus and dashboards that handle the numbers.

## Assets

**Kenney Minigolf Kit** (CC0, free): https://kenney.nl/assets/minigolf-kit
- 125 tiles: fairway straight/curve, greens, tees, bunkers, water, rough, paths, trees
- Color variations → cosmetic course themes (grass/desert/snow/links)

**Kenney Sports Pack** (CC0, free): https://kenney.nl/assets/sports-pack
- Golfer character sprites (top-down), flag pins, ball, bag

These provide the entire visual language. The management UI is pure SwiftUI.

## Core Loop

```
BUILD → SIMULATE → EARN → IMPROVE → REPUTATION GROWS
```

1. **BUILD**: Place/upgrade holes and facilities on your course grid
2. **SIMULATE**: Time advances automatically (1 real second = 1 game hour); members arrive, play rounds, spend money
3. **EARN**: Revenue minus costs = weekly profit; deposit to your improvement fund
4. **IMPROVE**: Spend earnings on new holes, better facilities, more staff
5. **REPUTATION**: Club star rating grows with quality + member satisfaction; unlocks premium members and higher prices

## Time System

- Time advances in **game weeks** (10 real seconds per game week at Normal speed)
- Speed controls: ⏸ Pause / ▶ Normal / ⏩ Fast (3×) / ⏭ Max (10×)
- 12 weeks = 1 game season (Spring/Summer/Autumn/Winter)
- Weather modifies play: Summer = peak; Winter = 60% capacity
- Player can take actions any time (pause optional; game waits for critical decisions)
- Week-end summary: shows weekly P&L, events, member changes

## Course Map

- **Grid**: 20×20 cells; each cell = one plot
- **View**: Top-down tile grid using Kenney sprites as SwiftUI `Image` assets
- **Starter plots**: 9 hole-plots unlocked; rest locked behind milestones
- **Tile types**:

| Tile | Purpose |
|------|---------|
| Tee Box | Hole start; upgradeable (rough → standard → manicured) |
| Fairway Straight/Curve | Path tiles; condition affects hole rating |
| Green | Hole end; upgradeable; most impactful on hole quality |
| Bunker | Hazard; increases hole challenge + appeal |
| Water Hazard | Hazard; high appeal, costs extra to maintain |
| Rough | Boundary filler; low cost |
| Trees/Foliage | Decoration; improves aesthetics score |
| Path | Connects clubhouse to holes; required for operation |
| Parking Lot | Required facility; capacity limits daily visitors |
| Clubhouse | Central building; upgradeable; revenue hub |
| Pro Shop | Revenue + member satisfaction |
| Restaurant/Bar | Revenue + member retention |
| Practice Range | Attract beginners; lesson revenue |
| Cart Barn | Optional; premium member perk |

## Holes

Each hole is a cluster of tiles (tee + fairway + green + optional hazards):

```swift
struct Hole {
    let number: Int
    var par: Int           // 3, 4, or 5
    var length: HoleLength // short/medium/long
    var teeGrade: Grade    // rough/standard/manicured/championship
    var fairwayGrade: Grade
    var greenGrade: Grade
    var hazards: [HazardType]
    var qualityRating: Double  // computed: 0.0–5.0
    var appealRating: Double   // computed; affects member satisfaction
}
```

Quality formula: `(tee.weight + fairway.weight + green.weight + hazardBonus) / 4`

## Members

| Tier | Monthly Dues | Green Fee | Expectation |
|------|-------------|-----------|-------------|
| Casual | $25 | $15 | Basic facilities |
| Regular | $60 | $0 (included) | Good course condition |
| Premium | $150 | $0 | Great course + facilities |
| Elite | $400 | $0 | Championship quality everything |

- New members join when: reputation reaches tier threshold AND satisfaction avg is high
- Members churn when satisfaction falls below their threshold for 2+ consecutive weeks
- Member count is the key long-term metric; green fee visitors are short-term revenue

## Staff

| Role | Cost/Week | Effect |
|------|-----------|--------|
| Groundskeeper | $300 | Maintains course condition; 1 needed per 9 holes |
| Pro Shop Staff | $200 | Pro shop revenue × 1.3 per staff member |
| F&B Staff | $250 | Restaurant revenue × 1.4 per staff member |
| Caddie | $150 | Attracts Premium/Elite members; satisfaction +5% |
| Golf Pro | $500 | Weekly lesson revenue ($400/pro); improves casual→regular conversion |
| Manager | $800 | Reduces member churn by 20%; required for Elite tier |

Understaffing consequences:
- No groundskeeper → course condition degrades → member satisfaction drops
- No F&B staff → restaurant closed → satisfaction drops for Premium/Elite

## Finances

**Revenue streams:**
- Green fees (visitors × daily fee)
- Membership dues (monthly; collected weekly pro-rata)
- Pro shop sales (~$20/member/week × pro shop grade multiplier)
- Restaurant/bar (~$15/member/week × F&B grade multiplier)
- Lesson fees (Golf Pro × $400/week)
- Event fees (see Events)

**Costs:**
- Staff salaries (weekly)
- Course maintenance base ($100/hole/week)
- Facility utilities ($50/facility/week)
- Capital improvement loans (optional; 5% weekly interest until repaid)

**Starting capital:** $10,000

## Progression & Milestones

| Milestone | Reward |
|-----------|--------|
| 25 members | Unlock Practice Range plot |
| 50 members | Unlock holes 10–12 |
| 1st tournament | +100 reputation, unlock Elite tier |
| 100 members | Unlock holes 13–15 |
| 5-star rating | Unlock Championship scenario |
| 200 members | Unlock holes 16–18 |
| Debt-free + 150 members | Win condition for campaign scenario |

## Events

Scheduled (player chooses):
- **Weekend Open**: Open to visitors; +50 visitors that weekend; costs $500 to run
- **Corporate Day**: Block booking; flat $2,000 revenue; members unavailable that day
- **Junior Clinic**: Led by Golf Pro; $300 income; +casual member conversion
- **Club Championship**: Requires 18 holes + championship grade greens; major reputation boost

Random (1–3 per season):
- **Storm damage**: Random hole condition drops 2 grades; repair costs $800
- **Staff resignation**: Lose a random staff member next week
- **Local press coverage**: +20 reputation boost (positive article)
- **Equipment failure**: Maintenance vehicle breaks; groundskeeping halted for 1 week ($600 repair)
- **Water main issue**: Water hazard tiles closed for 2 weeks

## Reputation System

`reputation = (avgHoleQuality × 0.4) + (facilitiesScore × 0.3) + (memberSatisfaction × 0.3)`

Maps to star rating:
- ⭐ 0–20: Struggling local course
- ⭐⭐ 21–40: Respectable community club
- ⭐⭐⭐ 41–60: Well-regarded regional club
- ⭐⭐⭐⭐ 61–80: Destination course
- ⭐⭐⭐⭐⭐ 81–100: Championship venue

Higher reputation → higher green fee ceiling → better member quality available

## Views / Screens

| View | Purpose |
|------|---------|
| `CourseMapView` | Scrollable 20×20 grid; tap to place/upgrade tiles; member sprites move around |
| `DashboardView` | Weekly P&L summary, member count, satisfaction, reputation |
| `FinancesView` | Revenue breakdown, costs, loan management, projections |
| `StaffView` | Hire/fire staff, assign roles, see morale |
| `MembersView` | Member roster, tier breakdown, satisfaction scores, churn risk |
| `UpgradesView` | Unlock new holes/facilities; upgrade existing tiles |
| `EventsView` | Schedule events; see upcoming random events; history |
| `SettingsView` | Speed controls, theme, sound, haptics, reset |

Navigation: `TabView` with icons for each main view (Dashboard, Course, Staff, Finances, Events).

## Monetization

**Free tier:** Full game — all 9-hole mechanics, all management systems, Zen (endless) mode

**IAP (StoreKit 2):**
- **18-Hole Expansion** ($4.99): Unlocks holes 10–18 + campaign win condition + Championship events
- **Desert Links Theme** ($1.99): Sandy/terracotta tile set from Kenney color variants
- **Mountain Course Theme** ($1.99): Snow/rock tile set
- **Ocean Links Theme** ($1.99): Blue/coastal tile set

No ads. No energy. No pay-to-win. Cosmetics and content expansion only.

## Tech Stack

Follows **APP_SCAFFOLD.md Part 3 (Utility Extension)** with game-like simulation layer:

| Component | Choice |
|-----------|--------|
| UI framework | SwiftUI |
| Entry point | `@main App` struct |
| Data layer | SwiftData + CloudKit |
| Platform | iOS 17.0+ |
| Simulation | `@Observable SimulationEngine` driven by `Timer.publish` |
| Course view | SwiftUI `LazyVGrid` + Kenney `Image` assets |
| IAP | StoreKit 2 |
| Audio | AVAudioEngine (synthesized) |

No SpriteKit. The "simulation" is a timer-based state machine, not physics.

## Data Models (SwiftData)

```swift
@Model class Club {
    var name: String
    var foundedDate: Date
    var cash: Double
    var reputation: Double
    var memberCount: Int
    var currentSeason: Season
    var currentWeek: Int
    var holes: [Hole]
    var facilities: [Facility]
    var staff: [StaffMember]
    var members: [ClubMember]
    var financialHistory: [WeeklyFinancials]
}

@Model class Hole { ... }
@Model class Facility { ... }
@Model class StaffMember { ... }
@Model class ClubMember { ... }
@Model class WeeklyFinancials { ... }
```

## Simulation Engine

```swift
@Observable
final class SimulationEngine {
    static let shared = SimulationEngine()

    var club: Club?
    var isPaused: Bool = false
    var speed: SimSpeed = .normal  // .paused / .normal / .fast / .max

    func tick(deltaWeeks: Double) {
        // 1. Advance time
        // 2. Apply weather modifier
        // 3. Calculate visitors and member plays
        // 4. Generate revenue
        // 5. Deduct costs
        // 6. Update member satisfaction
        // 7. Apply member churn and acquisition
        // 8. Check random events
        // 9. Update reputation
        // 10. Trigger milestone checks
        // 11. Persist via SwiftData
    }
}
```

## Persistence Keys (UserDefaults — non-SwiftData)

```swift
enum GreenKeeperKeys {
    static let soundEnabled       = "gk.soundEnabled"
    static let hapticsEnabled     = "gk.hapticsEnabled"
    static let currentTheme       = "gk.currentTheme"
    static let sawTutorial        = "gk.sawTutorial"
    static let speedSetting       = "gk.speedSetting"
    static let unlockedThemes     = "gk.unlockedThemes"
    static let hasPurchasedExpansion = "gk.hasPurchasedExpansion"
}
```

## Development Checklist

From `APP_SCAFFOLD.md` Utility Extension (U10):

- [ ] Define data models — Club, Hole, Facility, StaffMember, ClubMember, WeeklyFinancials
- [ ] Build `SimulationEngine` — tick loop, all revenue/cost calculations, event system
- [ ] Build `CourseMapView` — scrollable grid, Kenney tile images, member sprites
- [ ] Build `DashboardView` — weekly summary, reputation, member count
- [ ] Build `FinancesView` — P&L breakdown, charts, loan management
- [ ] Build `StaffView` — hire/fire, role assignment
- [ ] Build `MembersView` — roster, satisfaction, churn risk
- [ ] Build `UpgradesView` — tile upgrades, new plots
- [ ] Build `EventsView` — scheduling, history, random event display
- [ ] Build `SettingsView` — speed, theme, sound, haptics, reset
- [ ] Add iCloud sync — SwiftData + CloudKit
- [ ] Add onboarding — tutorial walkthrough of first 3 weeks
- [ ] Implement StoreKit 2 — expansion + 3 cosmetic themes
- [ ] Integrate Kenney assets — import sprites, map to tile types + themes
- [ ] Generate app icon — `generate_icon.swift` (green flag on dark background)
