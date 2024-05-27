[![FIWARE Banner](https://fiware.github.io/tutorials.Accessing-Context/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Accessing-Context.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)

This tutorial teaches FIWARE users how to alter the context programmatically. The tutorial builds on the entities
created in the previous [stock management example](https://github.com/FIWARE/tutorials.Context-Providers/) and enables a
user understand how to write code in an [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) capable
[Node.js](https://nodejs.org/) [Express](https://expressjs.com/) application in order to retrieve and alter context
data. This removes the need to use the command-line to invoke cUrl commands.

The tutorial is mainly concerned with discussing code written in Node.js, however some of the results can be checked by
making [cUrl](https://ec.haxx.se/) commands.

# Start-Up

**NGSI-v2** offers JSON based interoperability used in individual Smart Systems. To run this tutorial with **NGSI-v2**, use the `NGSI-v2` branch.

```console
git clone https://github.com/FIWARE/tutorials.Accessing-Context.git
cd tutorials.Accessing-Context
git checkout NGSI-v2

./services create
./services start
```


| [![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/) | :books: [Documentation](https://github.com/FIWARE/tutorials.Accessing-Context/tree/NGSI-v2) |
| --- | --- |

---

## License

[MIT](LICENSE) © 2018-2024 FIWARE Foundation e.V.
