
# Property Services Taxonomy

## Work Types
- **Repair**: Fix existing damage or failures.
- **Maintenance**: Preventative or routine servicing.
- **Improvement / Renovation**: Upgrades, remodels, replacements, or new installations.
- **Inspection**: Assessment, diagnostics, compliance.
- **Emergency**: Urgent response.

## Trade Catalog

### General Contracting
| Service | Work Type |
|---|---|
| Home renovations | Improvement |
| Commercial renovations | Improvement |
| Additions | Improvement |
| Tenant improvements | Improvement |
| Kitchen remodels | Improvement |
| Bathroom remodels | Improvement |
| New construction | Improvement |
| Project management | Improvement |

### Roofing
| Service | Work Type |
| Roof inspection | Inspection |
| Roof repair | Repair |
| Roof replacement | Improvement |
| Flat/Metal/Shingle/Tile roofing | Improvement |
| Roof coating | Maintenance |
| Skylight installation | Improvement |
| Gutters & downspouts | Repair, Maintenance, Improvement |

### Plumbing
| Service | Work Type |
| Leak repair | Repair |
| Pipe replacement | Repair, Improvement |
| Water heaters | Repair, Improvement |
| Sewer repair | Repair |
| Drain cleaning | Maintenance |
| Fixtures | Improvement |
| Gas lines | Repair, Improvement |
| Water filtration | Improvement |
| Commercial plumbing | Repair, Maintenance, Improvement |

### Electrical
| Service | Work Type |
| Wiring/Rewiring | Repair, Improvement |
| Panel upgrades | Improvement |
| Breakers | Repair |
| Lighting | Improvement |
| EV chargers | Improvement |
| Generators | Improvement |
| Smart home | Improvement |
| Security/Fire alarms | Improvement, Maintenance |

### HVAC
| Service | Work Type |
| AC/Furnace repair | Repair |
| AC/Furnace installation | Improvement |
| Heat pumps/Mini splits | Improvement |
| Ductwork | Repair, Improvement |
| Refrigeration | Repair |
| Preventative maintenance | Maintenance |

### Carpentry
Framing (Improvement), Finish carpentry (Improvement), Decks (Improvement), Doors & Windows (Repair/Improvement), Trim & Built-ins (Improvement)

### Drywall
Installation (Improvement), Repair (Repair), Texture matching (Repair), Ceiling repair (Repair), Water damage repair (Repair)

### Painting
Interior/Exterior painting (Improvement), Cabinet refinishing (Improvement), Epoxy coatings (Improvement), Pressure washing (Maintenance), Staining (Improvement)

### Flooring
Hardwood, Vinyl, Laminate, Tile, Carpet installation (Improvement), Floor refinishing (Improvement), Concrete polishing (Improvement)

### Concrete & Masonry
Driveways, Patios, Sidewalks, Retaining walls (Improvement), Foundation repair (Repair), Brick/Stone/Chimney repair (Repair), Parking lots (Improvement)

### Foundation & Waterproofing
Foundation repair (Repair), Basement waterproofing (Improvement), Crawlspace encapsulation (Improvement), French drains (Improvement), Sump pumps (Improvement), Crack repair (Repair)

### Glass & Windows
Window repair (Repair), Window replacement (Improvement), Storefront glass (Repair/Improvement), Shower doors & Mirrors (Improvement)

### Cabinets & Countertops
Cabinets, Quartz, Granite, Marble, Butcher block, Commercial millwork (Improvement)

### Landscaping
Lawn care (Maintenance), Irrigation (Maintenance/Improvement), Drainage (Improvement), Sod & Mulch (Improvement), Hardscaping (Improvement), Outdoor lighting (Improvement)

### Tree Services
Tree trimming (Maintenance), Tree removal (Maintenance/Emergency), Stump grinding (Maintenance), Lot clearing (Improvement)

### Fencing
Fence installation (Improvement), Fence repair (Repair), Gates & Automation (Improvement)

### Restoration
Water, Fire, Smoke, Storm damage restoration (Repair/Emergency), Mold remediation (Repair), Biohazard cleanup (Emergency)

### Environmental
Mold inspection (Inspection), Mold remediation (Repair), Asbestos removal (Repair), Lead removal (Repair), Radon mitigation (Improvement), IAQ testing (Inspection)

### Cleaning & Maintenance
Pressure washing, Window cleaning, Gutter cleaning, Air duct cleaning, Dryer vent cleaning, Chimney cleaning, Janitorial (Maintenance)

### Pest Control
Termites, Rodents, General pests, Wildlife, Mosquitoes (Maintenance/Repair)

### Specialty Commercial
Elevators, Loading docks, Commercial refrigeration, Restaurant equipment, Automatic doors, Signs (Repair, Maintenance, Improvement)

### Inspection Services
Home, Commercial, Roof, Foundation, HVAC, Electrical, Plumbing, Energy, Insurance (Inspection)

## Suggested Database

```text
Trades
- TradeId
- Name

Services
- ServiceId
- TradeId
- Name
- WorkType

Contractors
- ContractorId
- Company
- License

ContractorTrades
- ContractorId
- TradeId

ContractorServices
- ContractorId
- ServiceId
```

This model lets a contractor belong to multiple trades and offer many services while AI maps detected issues to services and then to qualified contractors.
