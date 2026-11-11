import { readFileSync } from "node:fs";
import { emitKeypressEvents } from "node:readline";


const TEXT = `install :
DBeaver, XAMPP, Nodejs,vs code, pnpm
https://code.visualstudio.com/download?_exp_download=d53503e735
https://www.apachefriends.org/de/download.html
https://nodejs.org/en/download
https://dbeaver.io/download/
npm install -g pnpm

extntions in vs.code : 
Prettier ESlint
Pretty Typescript Errors
REST Clinet

node documents herunterladen : 

Invoke-WebRequest \` -Uri "https://nodejs.org/download/release/latest-v24.x/docs/api/all.html" \`-OutFile "docs\nodejs\all.html"


express docus herunterladen : 

https://github.com/expressjs/expressjs.com

mantine dokus : 
Invoke-WebRequest -Uri "https://mantine.dev/llms-full.txt" -OutFile "mantine-doku.txt"

qss : 
https://coldrockgames.itch.io/qss/download/rjL_5AxQzJu1pWTMBTyl66uvF7IEepJnJeSZyq2W


erstell project folder
mkdir frontend. backend, docs 

_________
Frontend : 

pnpm create vite . --template react-ts  (need internet)
pnpm install   (need internet)
pnpm add @mantine/core @mantine/hooks
pnpm add @mantine/form
_________
in backend : 
pnpm init
pnpm add express mysql2
pnpm add -D typescript @types/node @types/express eslint @eslint/js typescript-eslint
pnpm exec tsc --init

in tsconfig.json  (compilerOptions  from express)  

____

in backend mkdir src/server.ts

in server.ts (express, hello-world, mjs variant, dont forget to add app.use... from debugging file )



____

in backend/package.json : 
"type": "module",
"scripts": {
  "dev": "node --watch src/server.ts",
  "check": "tsc --noEmit",
  "start": "node src/server.ts"
}


_____



DB in DBeaver erstellen


in server.ts die requests get,post,delete und put schreiben

_______

frontend : 

main.tsx :  add MantineProvider and his styles.css
vite.config.ts : add      
server: {
    proxy: {
      "/dishes": "http://localhost:3000",
    },
  },
  
  ______________
  ______________
  
  "import express from "express";
import mysql from "mysql2/promise";

const app = express();
const port = 3000;
app.use(express.json());

const db = mysql.createPool({
  host: "localhost",
  user: "root",
  password: "",
  database: "pizzaria",
  port: 3306,
});

app.get("/", (req, res) => {
  res.send("Server läuft");
});

app.get("/dishes", async (req, res) => {
  try{
    const [rows] = await db.query("SELECT * FROM dish")
    res.json(rows);
  }catch (error){
   console.error(error);
    res.status(500).json({error: "Datenbankfehler"})
  }
});

app.post("/dishes", async (req, res) => {
  try{

    const {name, price} = req.body;

    const [result] = await db.execute(
      "INSERT INTO dish (name, price) VALUES (?,?)",
      [name,price]
    );
    res.status(201).json(result);
    
  }catch (error){
   console.error(error);
    res.status(500).json({error: "Datenbankfehler"})
  }
});

app.put("/dishes/:id", async (req, res) => {
  try {
    const { id } = req.params;
    const { name, price } = req.body;

  const [result] = await db.execute(
    "UPDATE dish SET name = ?, price = ? WHERE dishid = ?",
    [name, price, id]
  );

    res.json(result);
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: "Datenbankfehler" });
  }
});

app.delete("/dishes/:id", async (req, res) => {
  try {
    const { id } = req.params;

    const [result] = await db.execute(
      "DELETE FROM dish WHERE dishid = ?",
      [id]
    );
    res.json(result);
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: "Datenbankfehler" });
  }
});

app.listen(port, () => {
  console.log(\`Server läuft auf http://localhost:\${port}\`);
});
  
`;

function textReader(text) {
  if (!process.stdin.isTTY || !process.stdout.isTTY) {
    console.error("Bitte in einem interaktiven Terminal starten.");
    process.exitCode = 1;
    return;
  }

  const lines = text.replace(/^\uFEFF/, "").split(/\r\n|\n|\r/);
  let index = 0;

  const previousRawMode = Boolean(process.stdin.isRaw);

  function render() {
    console.clear();
    console.log(`${index + 1}-${lines.length}`);
    console.log(
      lines[index].replace(/[\x00-\x08\x0B-\x1F\x7F-\x9F]/g, "")
    );
  }

  function stop() {
    process.stdin.setRawMode(previousRawMode);
    process.stdin.off("keypress", onKey);
    process.stdout.off("resize", render);
    process.off("SIGINT", stop);
    process.off("SIGTERM", stop);
    process.stdin.pause();

    console.log("\nProgramm beendet.");
    process.exit(0);
  }

  function onKey(_input, key) {
    if (!key) return;

    if (key.ctrl && key.name === "c") {
      return stop();
    }

    const previousIndex = index;

    if (key.name === "up") {
      index = Math.max(0, index - 1);
    }

    if (key.name === "down") {
      index = Math.min(lines.length - 1, index + 1);
    }

    if (index !== previousIndex) {
      render();
    }
  }

  emitKeypressEvents(process.stdin);
  process.stdin.setRawMode(true);
  process.stdin.resume();

  process.stdin.on("keypress", onKey);
  process.stdout.on("resize", render);
  process.on("SIGINT", stop);
  process.on("SIGTERM", stop);

  render();
}

try {
  const text = process.argv[2]
    ? readFileSync(process.argv[2], "utf8")
    : TEXT;

  textReader(text);
} catch (error) {
  console.error(`Fehler: ${error.message}`);
  process.exitCode = 1;
}
