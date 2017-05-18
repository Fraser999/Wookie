# Examples of Fuzzily Ordered Chains

* [Scenario 1](https://fraser999.github.io/Wookie/Example%201): two drop and one joins concurrently - no merge/split regardless of order
* [Scenario 2](https://fraser999.github.io/Wookie/Example%202): one joins, causing a split
* [Scenario 3](https://fraser999.github.io/Wookie/Example%203): one drops, causing a merge
* [Scenario 4](https://fraser999.github.io/Wookie/Example%204): two drop concurrently, the first causing a merge
* [Scenario 5](https://fraser999.github.io/Wookie/Example%205): one drops and one joins concurrently, but if the drop accumulates first, the section merges
* [Scenario 6](https://fraser999.github.io/Wookie/Example%206): one partially drops from half its own section but is able to reconnect
* [Scenario 7](https://fraser999.github.io/Wookie/Example%207): version x for a section accumulates before section x-1
* [Scenario 8](https://fraser999.github.io/Wookie/Example%208): two neighbour sections split and/or merge concurrently
