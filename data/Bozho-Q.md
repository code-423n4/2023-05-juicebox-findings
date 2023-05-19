#1

Project's dependency `prb-math` in package.json file is using `^2.4.3` which isn't using the latest version.

#### Recommendation
Use the latest version.

====

#2

`import '@openzeppelin/contracts/access/Ownable.sol';` is an unused dependecy. No functions are privileged to the owner.