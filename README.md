import React, { useEffect, useState } from "react"; import { Card, CardContent } from "@/components/ui/card"; import { Button } from "@/components/ui/button"; import { Input } from "@/components/ui/input"; import { Search } from "lucide-react"; import { initializeApp } from "firebase/app"; import { getFirestore, doc, getDoc, setDoc, deleteDoc, collection, getDocs } from "firebase/firestore";

// ⚠️ DEMO ONLY SYSTEM const firebaseConfig = { apiKey: "YOUR_API_KEY", authDomain: "YOUR_AUTH_DOMAIN", projectId: "YOUR_PROJECT_ID", storageBucket: "YOUR_STORAGE_BUCKET", messagingSenderId: "YOUR_SENDER_ID", appId: "YOUR_APP_ID" };

const app = initializeApp(firebaseConfig); const db = getFirestore(app);

const ADMIN_PASSWORD = "Colonel@1";

export default function PermitSystem() { const [licenseNumber, setLicenseNumber] = useState(""); const [nephNumber, setNephNumber] = useState("98765432"); const [result, setResult] = useState(null); const [loading, setLoading] = useState(false);

const [isAdmin, setIsAdmin] = useState(false); const [password, setPassword] = useState("");

const [permits, setPermits] = useState([]); const [stats, setStats] = useState({ total: 0, b: 0, c: 0, other: 0 });

const [form, setForm] = useState({ nom: "", prenom: "", naissance: "", lieu: "", nationalite: "", dossier: "", categorie: "B", statut: "Enregistré" });

const fetchPermits = async () => { const snap = await getDocs(collection(db, "permis")); const data = snap.docs.map(d => ({ id: d.id, ...d.data() }));

setPermits(data);

// 📊 STATS CALCULATION
const total = data.length;
const b = data.filter(p => p.categorie === "B").length;
const c = data.filter(p => p.categorie === "C").length;
const other = total - b - c;

setStats({ total, b, c, other });

};

useEffect(() => { if (isAdmin) fetchPermits(); }, [isAdmin]);

const login = () => { if (password === ADMIN_PASSWORD) setIsAdmin(true); else alert("Mot de passe incorrect"); };

const verify = async () => { setLoading(true);

const ref = doc(db, "permis", nephNumber);
const snap = await getDoc(ref);

if (snap.exists()) {
  const d = snap.data();
  setResult(`Permis: ${d.nom} ${d.prenom} | NEPH: ${nephNumber} | Catégorie: ${d.categorie} | Statut: ${d.statut}`);
} else {
  setResult("Aucun permis trouvé");
}

setLoading(false);

};

const addPermit = async () => { const neph = String(Math.floor(10000000 + Math.random() * 90000000));

await setDoc(doc(db, "permis", neph), {
  ...form,
  neph
});

setForm({ nom: "", prenom: "", naissance: "", lieu: "", nationalite: "", dossier: "", categorie: "B", statut: "Enregistré" });

fetchPermits();

};

const removePermit = async (id) => { await deleteDoc(doc(db, "permis", id)); fetchPermits(); };

return ( <div className="min-h-screen bg-slate-100 p-6">

<header className="bg-blue-900 text-white p-6 rounded-xl mb-6">
    <div className="text-xs">DEMO SYSTEM</div>
    <h1 className="text-xl font-bold">Permis Dashboard</h1>
  </header>

  {!isAdmin ? (
    <Card className="max-w-md mx-auto">
      <CardContent className="p-4 space-y-3">
        <h2>Connexion Admin</h2>

        <Input type="password" placeholder="Mot de passe" value={password} onChange={e => setPassword(e.target.value)} />
        <Button onClick={login}>Connexion</Button>

        <hr />

        <Input placeholder="NEPH" value={nephNumber} onChange={e => setNephNumber(e.target.value)} />
        <Input placeholder="Numéro permis" value={licenseNumber} onChange={e => setLicenseNumber(e.target.value)} />
        <Button onClick={verify}>Vérifier</Button>

        {result && <div className="p-2 bg-white border mt-2">{result}</div>}
      </CardContent>
    </Card>
  ) : (
    <div className="grid md:grid-cols-2 gap-4">

      {/* STATS */}
      <Card>
        <CardContent className="p-4">
          <h2 className="font-bold mb-2">Statistiques</h2>
          <p>Total permis : {stats.total}</p>
          <p>Catégorie B : {stats.b}</p>
          <p>Catégorie C : {stats.c}</p>
          <p>Autres : {stats.other}</p>
        </CardContent>
      </Card>

      {/* ADD */}
      <Card>
        <CardContent className="p-4 space-y-2">
          <h2>Ajouter permis</h2>
          {

["nom","prenom","naissance","lieu","nationalite","dossier","categorie","statut"].map((k) => ( k === "categorie" ? ( <select key={k} value={form[k]} onChange={(e) => setForm({ ...form, [k]: e.target.value })} className="border p-2 w-full rounded" > <option value="AM">AM - Cyclomoteur</option> <option value="A1">A1 - Moto légère</option> <option value="A2">A2 - Moto intermédiaire</option> <option value="A">A - Toutes motos</option> <option value="B">B - Voiture</option> <option value="B1">B1 - Quadricycle</option> <option value="BE">BE - Voiture + remorque</option> <option value="B96">B96 - Extension remorque</option> <option value="C1">C1 - Petit camion</option> <option value="C">C - Poids lourd</option> <option value="C1E">C1E - Petit camion + remorque</option> <option value="CE">CE - Poids lourd + remorque</option> <option value="D1">D1 - Petit bus</option> <option value="D">D - Bus</option> <option value="D1E">D1E - Petit bus + remorque</option> <option value="DE">DE - Bus + remorque</option> </select> ) : ( <Input key={k} placeholder={k} value={form[k]} onChange={(e) => setForm({ ...form, [k]: e.target.value })} /> ) )) } <Button onClick={addPermit}>Ajouter</Button> </CardContent> </Card>

{/* LIST */}
      <Card className="md:col-span-2">
        <CardContent className="p-4">
          <h2>Liste permis</h2>
          {permits.map(p => (
            <div key={p.id} className="border p-2 mb-2 rounded">
              {p.nom} {p.prenom} - {p.categorie} - {p.neph}
              <Button onClick={() => removePermit(p.id)} className="ml-2">Supprimer</Button>
            </div>
          ))}
        </CardContent>
      </Card>

    </div>
  )}

</div>

); }
