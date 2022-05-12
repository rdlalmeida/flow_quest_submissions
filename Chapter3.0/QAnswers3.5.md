Area 1:

|| Variable a | Variable b | Variable c | Variable d |
|:-----------|:----------:|:----------:|:----------:|:----------:|
| Read | &#x2713; | &#x2713; | &#x2713; | &#x2713; |
| Write| &#x2713; | &#x2713; | &#x2713; | &#x2713; |

Area 2:
This is a curious one... Apparently it is impossible to access any variables from a certain element (struct, resource, etc.) from within another element in the same contract, i.e, defining a new struct or resource inside a contract that already has another, the new instance is not allowed to know what was already defined inside that contract. Couldn't find any way to access the upper level (contract) from within the Resouce with my current Cadence level...

|| Variable a | Variable b | Variable c | Variable d |
|:-----------|:----------:|:----------:|:----------:|:----------:|
| Read | &#x2717; | &#x2717; | &#x2717; | &#x2717; |
| Write| &#x2717; | &#x2717; | &#x2717; | &#x2717; |

Area 3:

|| Variable a | Variable b | Variable c | Variable d |
|:-----------|:----------:|:----------:|:----------:|:----------:|
| Read | &#x2713; | &#x2713; | &#x2713; | &#x2717; |
| Write| &#x2713; | &#x2717; | &#x2717; | &#x2717; |

Area 4:

|| Variable a | Variable b | Variable c | Variable d |
|:-----------|:----------:|:----------:|:----------:|:----------:|
| Read | &#x2713; | &#x2713; | &#x2717; | &#x2717; |
| Write| &#x2713; | &#x2717; | &#x2717; | &#x2717; |

And now for the functions

|| Area 1 | Area 2 | Area 3 | Area 4 |
|:--|:-:|:-:|:-:|:-:|
|publicFunc()   | &#x2713; | &#x2717; | &#x2713; | &#x2713; |
|contractFunc() | &#x2713; | &#x2717; | &#x2713; | &#x2717; |
|privateFunc()  | &#x2713; | &#x2717; | &#x2717; | &#x2717; |
