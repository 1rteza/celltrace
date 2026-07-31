const { MongoClient } = require("mongodb");
const admin = require("firebase-admin");

if (!admin.apps.length) {
  const serviceAccount = JSON.parse(process.env.FIREBASE_SERVICE_ACCOUNT_JSON);
  admin.initializeApp({
    credential: admin.credential.cert(serviceAccount),
  });
}

// Reuse the MongoDB connection across warm serverless invocations.
let cachedClientPromise = null;
function getClient() {
  if (!cachedClientPromise) {
    const client = new MongoClient(process.env.MONGODB_URI);
    cachedClientPromise = client.connect();
  }
  return cachedClientPromise;
}

async function verifyUid(authHeader) {
  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    throw new Error("missing-token");
  }
  const idToken = authHeader.slice(7);
  const decoded = await admin.auth().verifyIdToken(idToken);
  return decoded.uid;
}

function isUsableEntry(e) {
  return !!e && typeof e.percentage === "number" && !isNaN(e.percentage) && !!e.timestamp && !isNaN(new Date(e.timestamp).getTime());
}
function isUsableDevice(d) {
  return !!d && typeof d.id === "string" && typeof d.name === "string";
}

exports.handler = async (event) => {
  const headers = {
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Headers": "Content-Type, Authorization",
    "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
    "Content-Type": "application/json",
  };

  if (event.httpMethod === "OPTIONS") {
    return { statusCode: 204, headers, body: "" };
  }

  let uid;
  try {
    const authHeader = event.headers.authorization || event.headers.Authorization;
    uid = await verifyUid(authHeader);
  } catch (e) {
    return { statusCode: 401, headers, body: JSON.stringify({ error: "Unauthorized" }) };
  }

  let client;
  try {
    client = await getClient();
  } catch (e) {
    console.error("mongo connect failed", e);
    return { statusCode: 500, headers, body: JSON.stringify({ error: "Database connection failed" }) };
  }

  const collection = client.db("CellTrace").collection("backups");

  try {
    if (event.httpMethod === "GET") {
      const doc = await collection.findOne({ uid });
      const devices = doc && Array.isArray(doc.devices) ? doc.devices.filter(isUsableDevice) : [];
      const entries = doc && Array.isArray(doc.entries) ? doc.entries.filter(isUsableEntry) : [];
      return {
        statusCode: 200,
        headers,
        body: JSON.stringify({ devices, entries, updatedAt: doc ? doc.updatedAt : null }),
      };
    }

    if (event.httpMethod === "POST") {
      let parsed;
      try {
        parsed = JSON.parse(event.body || "{}");
      } catch (e) {
        return { statusCode: 400, headers, body: JSON.stringify({ error: "Invalid JSON body" }) };
      }
      const devices = (Array.isArray(parsed.devices) ? parsed.devices : []).filter(isUsableDevice);
      const entries = (Array.isArray(parsed.entries) ? parsed.entries : []).filter(isUsableEntry);
      const updatedAt = new Date().toISOString();
      await collection.updateOne(
        { uid },
        { $set: { uid, devices, entries, updatedAt } },
        { upsert: true }
      );
      return { statusCode: 200, headers, body: JSON.stringify({ devices, entries, updatedAt }) };
    }

    return { statusCode: 405, headers, body: JSON.stringify({ error: "Method not allowed" }) };
  } catch (e) {
    console.error("handler error", e);
    return { statusCode: 500, headers, body: JSON.stringify({ error: "Server error" }) };
  }
};
