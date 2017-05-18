# Scenario 3: one drops, causing a merge

`MIN_SECTION_SIZE` is 3.

start: `Block { pfx: 1100, v: 1314, mbrs: [C0-C3] }` and `Block { pfx: 1101, v: 1297, mbrs: [D0-D2] }`

`D2` leaves, causing section `1101` to have just 2 members.

#### Step 1 - Nodes `D0` and `D1`:
- see D2 drop, so send `vote_del_D2_from_1297: Vote { to: Block { pfx: 1101, v: 1298, mbrs: [D0, D1] }, from: Block { pfx: 1101, v: 1297, mbrs: [D0-D2] } }`
- Current Blocks: `[ {pfx: 1100, v: 1314, mbrs: [C0-C3]}, {pfx: 1101, v: 1297, mbrs: [D0-D2]} ]`

#### Step 2 - Nodes `C0` to `C3`:
- recv `D0/1`'s `vote_del_D2_from_1297` (accumulates)
    - since section `1101` has too few members, send `vote_merge: { to: Block { pfx: 110, v: 1315, mbrs: [C0-C3, D0, D1] }, from: Block { pfx: 1101, v: 1298, mbrs: [D0, D1] } }`
    - Current Blocks: `[ {pfx: 1100, v: 1314, mbrs: [C0-C3]}, {pfx: 1101, v: 1298, mbrs: [D0, D1]} ]`

#### Step 3 - Nodes `D0` and `D1`:
- recv `C0-C3`'s `vote_merge` (accumulates, but not yet valid)
- recv own `vote_del_D2_from_1297` (accumulates, becomes current briefly, then is succeeded by the merge vote)
    - while `vote_del_D2_from_1297` is current:
        - Current Blocks: `[ {pfx: 1100, v: 1314, mbrs: [C0-C3]}, {pfx: 1101, v: 1298, mbrs: [D0, D1]} ]`
        - send `vote_merge: { to: Block { pfx: 110, v: 1315, mbrs: [C0-C3, D0, D1] }, from: Block { pfx: 1101, v: 1298, mbrs: [D0, D1] } }`
    - now that the merge vote has become valid:
        - Current Blocks: `[ {pfx: 110, v: 1315, mbrs: [C0-C3, D0, D1]} ]`

#### Step 4 - Nodes `C0` to `C3`:
- recv all `vote_merge` (accumulates)
    - Current Blocks: `[ {pfx: 110, v: 1315, mbrs: [C0-C3, D0, D1]} ]`
