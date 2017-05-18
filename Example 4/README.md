# Scenario 4: two drop concurrently, the first causing a merge

`MIN_SECTION_SIZE` is 5.

start: `Block { pfx: 1100, v: 1314, mbrs: [C0-C4] }` and `Block { pfx: 1101, v: 1297, mbrs: [D0-D4] }`

`D3` and `D4` leave, causing section `1101` to have just two members.

#### Step 1 - Nodes `D0` to `D3`:
- see `D4` drop, so send `vote_del_D4_from_1297: Vote { to: Block { pfx: 1101, v: 1298, mbrs: [D0-D3] }, from: Block { pfx: 1101, v: 1297, mbrs: [D0-D4] } }`
- Current Blocks: `[ {pfx: 1100, v: 1314, mbrs: [C0-C4]}, {pfx: 1101, v: 1297, mbrs: [D0-D4]} ]`

#### Step 2 - Nodes `D0` to `D2`:
- see `D3` drop, so send `vote_del_D3_from_1297: Vote { to: Block { pfx: 1101, v: 1298, mbrs: [D0-D2, D4] }, from: Block { pfx: 1101, v: 1297, mbrs: [D0-D4] } }`
- Current Blocks: `[ {pfx: 1100, v: 1314, mbrs: [C0-C3]}, {pfx: 1101, v: 1297, mbrs: [D0-D3]} ]`

#### Step 3 - Nodes `C0` to `C4`:
- recv all `vote_del_D4_from_1297`
    - Current Blocks: `[ {pfx: 1100, v: 1314, mbrs: [C0-C4]}, {pfx: 1101, v: 1298, mbrs: [D0-D3]} ]`
    - will disconnect from `D4` in 1 minute since `D4` is missing from a current block
    - since sibling of ours has too few members, send `vote_merge_from_1298: Vote { to: Block { pfx: 110, v: 1315, mbrs: [C0-C4, D0-D3] }, from: Block { pfx: 1100, v: 1314, mbrs: [C0-C4] } }`

#### Step 4 - Nodes `D0` to `D2`:
- recv own `vote_del_D3_from_1297`
    - Current Blocks: `[ {pfx: 1100, v: 1314, mbrs: [C0-C4]}, {pfx: 1101, v: 1298, mbrs: [D0-D2, D4]} ]`
    - since `D4` is missing from a current block, send `vote_del_D4_from_1298: Vote { to: Block { pfx: 1101, v: 1299, mbrs: [D0-D2] }, from: Block { pfx: 1101, v: 1298, mbrs: [D0-D2, D4] } }`
    - since our current block has too few members, send `vote_merge_from_1298: Vote { to: Block { pfx: 110, v: 1315, mbrs: [C0-C4, D0-D2, D4] }, from: Block { pfx: 1101, v: 1298, mbrs: [D0-D2, D4] } }`

#### Step 5 - Nodes `C0` to `C4`:
- recv all `vote_del_D4_from_1298`
    -
