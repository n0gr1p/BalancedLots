# BalancedLots

Windower 4 multi-client addon for **evenly distributing a configured treasure-pool item**. Adapted from [KuponsLot](https://github.com/n0gr1p/KuponsLot), with thanks to its local IPC coordination and virtual allocation model.

## Default: VR 10-3 / 101 Dazbogs

The default item is **Fat Worm Meat**, and the per-character goal is **6**.

For a 6-character party, this means each client gets assigned one drop at a time toward a 6-item cap, until the party collectively has 36.

## What it does

1. Each loaded client watches its treasure pool for the configured item.
2. Nearby-party clients in the same zone exchange their actual inventory+storage totals and receiving capacity via Windower IPC.
3. A deterministic leader allocates each unclaimed drop to an enabled, eligible client with the lowest **virtual** count; multiple simultaneous drops reserve capacity and increment virtual totals as they are assigned.
4. **Only the selected character issues `windower.ffxi.lot_item(slot)`.** BalancedLots **never calls pass_item**. Existing pass-after-lot automation remains responsible for passing on other clients.
5. Acknowledged lots are not re-issued. Failed assignments can be retried or reassigned.
6. If `goal > 0`, clients that already have enough copies are excluded, and in-flight assignments do not exceed the goal.

Equal totals are broken deterministically. Disabled clients, observe-only clients when auto clients exist, and clients lacking main-inventory space are excluded.

## Setup

Put the folder at `Windower4/addons/BalancedLots/` with `BalancedLots.lua` inside. Load it on **all participating characters**:

```text
//lua l BalancedLots
```

The default target is already **Fat Worm Meat, six per character**. To inspect:

```text
//blot status
//blot totals
//blot peers
```

To change the item or target, enter these commands **on any one loaded client**. The addon broadcasts the configuration to other local BalancedLots clients and saves it on each:

```text
//blot item Fat Worm Meat
//blot goal 6
```

You can use an exact English item name or a numeric Windower item ID. `//blot goal 0` means unlimited / distribute as evenly as possible.

### Safe first test

Enable observe-only mode on **all six clients**, or use your usual Windower Send broadcast:

```text
//blot observe on
```

In observe mode, assignments are displayed but **no lots are issued**. Run `//blot peers` and `//blot status` after a drop. When allocations look right, turn observe mode off on each client:

```text
//blot observe off
```

Observe mode **does not** synchronize between clients (each client may opt out independently). `item` and `goal` changes **do** synchronize.

Other commands:

```text
//blot on
//blot off
//blot verbose on
//blot verbose off
```

## Tracked totals

Like KuponsLot, totals are calculated from Inventory, Mog Safe, Mog Safe 2, Storage, Mog Locker, Mog Satchel, Mog Sack and Mog Case, as available through Windower's item APIs.

Receive capacity is computed from free **Inventory** slots plus room in existing item stacks. It is capped by the configured per-character goal.

## Interactions and limitations

- Do **not** use another independent auto-lot addon for the same item. Your existing **pass-only after another account lots** workflow is fine.
- All local participating clients must run BalancedLots and use the same item and goal. Different settings are rejected during peer exchange.
- This release supports **one configured target item at a time**; change it at any time via `//blot item ...`.
- Only members whose clients are running the addon can be selected. IPC covers the same local Windower environment, not remote machines.
- This is an initial port of KuponsLot and **needs an in-game observe-mode test** before relying on automatic lots.

## Source

Derived from [KuponsLot](https://github.com/n0gr1p/KuponsLot) and retains its deterministic leader election, lot acknowledgments, retries, virtual allocation and inventory capacity logic.
