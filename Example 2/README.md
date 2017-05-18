# Scenario 2: one joins, causing a split

`MIN_SECTION_SIZE` is 3, and `SPLIT_BUFFER` is 1.

start: `Block { pfx: 110, v: 1314, mbrs: [C0-C3, D0-D2] }`

`D3` joins, causing section `110` to have 4 members in `1100` and 4 in `1101`.

#### Step 1 - Nodes `C0` to `C3`:
- recv `D3`'s successful resource proof responses
- handle `D3`'s resource proof timeout
    - send `NodeApproval` to `D3`
    - send `vote_add_D3: Vote { to: Block { pfx: 110, v: 1315, mbrs: [C0-C3, D0-D3] }, from: Block { pfx: 110, v: 1314, mbrs: [C0-C3, D0-D2] } }`
    - send `vote_split_to_C: Vote { to: Block { pfx: 1100, v: 1316, mbrs: [C0-C3] }, from: Block { pfx: 110, v: 1315, mbrs: [C0-C3, D0-D3] } }`
    - send `vote_split_to_D: Vote { to: Block { pfx: 1101, v: 1316, mbrs: [D0-D3] }, from: Block { pfx: 110, v: 1315, mbrs: [C0-C3, D0-D3] } }`
- recv all their own `vote_add_D3` and `vote_split_to_C` (they both accumulate)
    - Current Blocks: `[ {pfx: 110, v: 1315, mbrs: [C0-C3, D0-D3]}, {pfx: 1100, v: 1316, mbrs: [C0-C3]} ]`
    - No need to send split messages as they've already been sent

#### Step 2 - Nodes `D0` to `D2`:
- recv `C0-C3`'s `vote_split_to_C` and `vote_split_to_D` (they accumulate but neither are valid since we're missing the predecessor)
    - Current Blocks: `[ {pfx: 110, v: 1314, mbrs: [C0-C3, D0-D2]} ]`

#### Step 3 - Nodes `C0` to `C3`:
- recv their own `vote_split_to_D` (accumulates, so we've completed the split)
    - Current Blocks: `[ {pfx: 1100, v: 1316, mbrs: [C0-C3]} ]`

#### Step 4 - Nodes `D0` to `D2`:
- recv `C0-C3`'s `vote_add_D3` (accumulate, become current briefly, then are succeeded by the split votes)
    - while `vote_add_D3` is current:
        - Current Blocks: `[ {pfx: 110, v: 1315, mbrs: [C0-C3, D0-D3]} ]`
        - If they're connected to `D3`, upgrade to a RT contact, otherwise start connecting to `D3`
    - now that the split votes have become valid:
        - Current Blocks: `[ {pfx: 1100, v: 1316, mbrs: [D0-D3]} ]`
