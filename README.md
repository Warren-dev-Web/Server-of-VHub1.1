// server.js
import express from "express";
import multer from "multer";
import cors from "cors";
import path from "path";
import fs from "fs";

const app = express();
app.use(cors());
app.use("/uploads", express.static(path.join(process.cwd(), "uploads")));

// Crée uploads si pas existant
if(!fs.existsSync("uploads")) fs.mkdirSync("uploads");

const storage = multer.diskStorage({
  destination: function(req, file, cb){
    cb(null, "uploads/");
  },
  filename: function(req, file, cb){
    cb(null, Date.now() + path.extname(file.originalname));
  }
});
const upload = multer({ storage });

// Liste des vidéos
let videos = [];

// Upload vidéo
app.post("/upload", upload.single("video"), (req,res)=>{
  const { title } = req.body;
  const file = req.file;
  if(!file || !title) return res.status(400).send("Video ou titre manquant");

  videos.push({ id: videos.length+1, title, filename: file.filename, likes:0, dislikes:0, comments:[], uploader: req.body.uploader });
  res.send({ message:"Upload OK" });
});

// Like vidéo
app.post("/like", express.json(), (req,res)=>{
  const { id } = req.body;
  const vid = videos.find(v=>v.id===id);
  if(vid){ vid.likes++; res.send({likes:vid.likes}); }
  else res.status(404).send("Vidéo non trouvée");
});

// Dislike vidéo
app.post("/dislike", express.json(), (req,res)=>{
  const { id } = req.body;
  const vid = videos.find(v=>v.id===id);
  if(vid){ vid.dislikes++; res.send({dislikes:vid.dislikes}); }
  else res.status(404).send("Vidéo non trouvée");
});

// Ajouter commentaire
app.post("/comment", express.json(), (req,res)=>{
  const { id, user, text } = req.body;
  const vid = videos.find(v=>v.id===id);
  if(vid){ vid.comments.push({user,text}); res.send({comments:vid.comments}); }
  else res.status(404).send("Vidéo non trouvée");
});

// Récupérer vidéos
app.get("/videos", (req,res)=>res.json(videos));

app.listen(process.env.PORT || 3000, ()=>console.log("Serveur lancé"));
