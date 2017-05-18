# Scenario 1: two drop and one joins concurrently - no merge/split regardless of order

start: `Block { v: 1314, mbrs: [C0-C4, D0, D1] }`

`D0` and `D1` drop, `E0` joins.

#### Step 1 - Nodes `C0` and `C1`:
- Current Blocks: `[ {v: 1314, mbrs: [C0-C4, D0, D1]} ]`
- see `D0` drop, so send `vote_del_D0_from_1314: Vote { to: Block { v: 1315, mbrs: [C0-C4, D1] }, from: Block { v: 1314, mbrs: [C0-C4, D0, D1] } }`
- see `D1` drop, so send `vote_del_D1_from_1314: Vote { to: Block { v: 1315, mbrs: [C0-C4, D0] }, from: Block { v: 1314, mbrs: [C0-C4, D0, D1] } }`

#### Step 2 - Nodes `C2` and `C3`:
- Current Blocks: `[ {v: 1314, mbrs: [C0-C4, D0, D1]} ]`
- pass `E0`, so send `vote_add_E0_to_1314: Vote { to: Block { v: 1315, mbrs: [C0-C4, D0, D1, E0] }, from: Block { v: 1314, mbrs: [C0-C4, D0, D1] } }`
- recv `C0` and `C1`'s `vote_del_D0_from_1314`
- see `D1` drop, so send `vote_del_D1_from_1314: Vote { to: Block { v: 1315, mbrs: [C0-C4, D0] }, from: Block { v: 1314, mbrs: [C0-C4, D0, D1] } }`

#### Step 3 - Node `C4`:
- Current Blocks: `[ {v: 1314, mbrs: [C0-C4, D0, D1]} ]`
- recv `C0/1`'s and `C2/3`'s `vote_del_D1_from_1314` (accumulates drop of `D1`)
    - Current Blocks: `[ {v: 1315, mbrs: [C0-C4, D0]} ]`
    - will disconnect from `D1` in 1 minute since `D1` is missing from a current block
- passes `E0`, so sends `vote_add_E0_to_1315: Vote { to: Block { v: 1316, mbrs: [C0-C4, D0, E0] }, from: Block { v: 1315, mbrs: [C0-C4, D0] } }`

#### Step 4 - Nodes `C0` and `C1`:
- recv `C2/3`'s `vote_add_E0_to_1314`
- recv `C4`'s `vote_add_E0_to_1315`
- recv `C2/3`'s `vote_del_D1_from_1314`
- recv `C0/1`'s `vote_del_D0_from_1314`
- recv `C0/1`'s `vote_del_D1_from_1314` (accumulates drop of `D1`)
    - Current Blocks: `[ {v: 1315, mbrs: [C0-C4, D0]} ]`
    - are disconnected from `D0`, so send `vote_del_D0_from_1315: Vote { to: Block { v: 1316, mbrs: [C0-C4] }, from: Block { v: 1315, mbrs: [C0-C4, D0] } }`

#### Step 5 - Nodes `C2` and `C3`:
- see `D0` drop, so send `vote_del_D0_from_1314: Vote { to: Block { v: 1315, mbrs: [C0-C4, D1] }, from: Block { v: 1314, mbrs: [C0-C4, D0, D1] } }`
- recv this^ (accumulates drop of `D0`)
    - Current Blocks: `[ {v: 1315, mbrs: [C0-C4, D1]} ]`
    - have `E0` as passed candidate, so send `vote_add_E0_to_1315: Vote { to: Block { v: 1316, mbrs: [C0-C4, D1, E0] }, from: Block { v: 1315, mbrs: [C0-C4, D1] } }`
    - are disconnected from `D1`, so send `vote_del_D1_from_1315: Vote { to: Block { v: 1316, mbrs: [C0-C4] }, from: Block { v: 1315, mbrs: [C0-C4, D1] } }`

#### Step 6 - Nodes `C0` and `C1`:
- recv `C0/1`'s `vote_del_D0_from_1315`
- recv `C2/3`'s `vote_del_D1_from_1315` (doesn't accumulate as their "from" Block has `[C0-C4, D1]` and `C0/1`'s "from" Block has `[C0-C4, D0]`)
- recv `C2/3`'s `vote_del_D0_from_1314` (accumulates drop of `D0`)
    - Current Blocks: `[ {v: 1315, mbrs: [C0-C4, D0]}, { v: 1315, mbrs: [C0-C4, D1] } ]`
    - are disconnected from `D0` and `D1`, so send `vote_del_D1_from_1315: Vote { to: Block { v: 1316, mbrs: [C0-C4] }, from: Block { v: 1315, mbrs: [C0-C4, D1] } }` (already sent `vote_del_D0_from_1315`)

#### Step 7 - Node `C4`:
- `D1` has now disconnected, but wasn't in a current block, so no vote sent
- recv `C2/3`'s `vote_add_E0_to_1314`
- recv own `vote_add_E0_to_1315`
- recv `C2/3`'s `vote_add_E0_to_1315`
- recv `C0/1`'s `vote_del_D0_from_1315`
- recv `C0-3`'s `vote_del_D1_from_1315` (accumulates but currently invalid as we don't have this version 1315; our only current blocks is `[ {v: 1315, mbrs: [C0-C4, D0]} ]`)

#### Step 8 - Nodes `C0` and `C1`:
- recv own `vote_del_D1_from_1315` (accumulates drop of `D1`)
    - Current Blocks: `[ { v: 1316, mbrs: [C0-C4] } ]`

#### Step 9 - Nodes `C2` and `C3`
- recv own `vote_add_E0_to_1314`
- recv `C0-3`'s `vote_del_D1_from_1314` (accumulates and is valid but not current, so doesn't trigger sending a vote)
- recv `C4`'s `vote_add_E0_to_1315`
- recv own `vote_add_E0_to_1315`
- recv `C0-3`'s `vote_del_D1_from_1315` (accumulates drop of `D1`)
    - Current Blocks: `[ {v: 1316, mbrs: [C0-C4]} ]`
    - have `E0` as passed candidate, so send `vote_add_E0_to_1316: Vote { to: Block { v: 1317, mbrs: [C0-C4, E0] }, from: Block { v: 1316, mbrs: [C0-C4] } }`

#### Step 10 - Node `C4`:
- recv `C0-3`'s `vote_del_D0_from_1314` (accumulates and makes invalid block now valid)
    - Before applying newly-valid block:
        - Current Blocks: `[ {v: 1315, mbrs: [C0-C4, D0]}, {v: 1315, mbrs: [C0-C4, D1]} ]`
        - will disconnect from `D0` in 1 minute since `D0` is missing from a current block
        - is now disconnected from `D1`, so sends `vote_del_D1_from_1315: Vote { to: Block { v: 1316, mbrs: [C0-C4] }, from: Block { v: 1315, mbrs: [C0-C4, D1] } }`
        - has `E0` as passed candidate, so sends `vote_add_E0_to_1315: Vote { to: Block { v: 1316, mbrs: [C0-C4, D1, E0] }, from: Block { v: 1315, mbrs: [C0-C4, D1] } }` (already sent `vote_add_E0_to_1315` for the other v1315)
    - Now applies buffered accumulated vote `vote_del_D1_from_1315: Vote { to: Block { v: 1316, mbrs: [C0-C4] }, from: Block { v: 1315, mbrs: [C0-C4, D1] } }`
        - Current Blocks: `[ {v: 1316, mbrs: [C0-C4]} ]`
        - keeps disconnect timer running for `D0` since `D0` is still missing from a current block (albeit a different one)
        - has `E0` as passed candidate, so sends `vote_add_E0_to_1316: Vote { to: Block { v: 1317, mbrs: [C0-C4, E0] }, from: Block { v: 1316, mbrs: [C0-C4] } }`

#### Step 11 - All Nodes:
- recv `C2-4`'s `vote_add_E0_to_1316`
    - Current Blocks: `[ { v: 1317, mbrs: [C0-C4, E0] } ]`
    - Nodes `C0` & `C1` will attempt to connect to `E0`
    - Nodes `C2-C4` upgrade candidate `E0` to full node
    - All send `NodeApproval` to `E0`
