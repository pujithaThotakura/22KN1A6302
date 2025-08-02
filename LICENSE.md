import React, { useState, useEffect } from "react";
function App() {
  const [longUrl, setLongUrl] = useState("");
  const [shortUrls, setShortUrls] = useState(
    () => JSON.parse(localStorage.getItem("shortUrls")) || {}
  );
  const [clickStats, setClickStats] = useState(
    () => JSON.parse(localStorage.getItem("clickStats")) || {}
  );
  const [shortUrl, setShortUrl] = useState("");

  const baseUrl = window.location.origin + "/#/";

useEffect(() => {
    localStorage.setItem("shortUrls", JSON.stringify(shortUrls));
    localStorage.setItem("clickStats", JSON.stringify(clickStats));
  }, [shortUrls, clickStats]);

  const generateKey = () => {
    const chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    let key = "";
    for (let i = 0; i < 5; i++) {
      key += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return key;
  };

  const handleShorten = () => {
    if (!longUrl.trim()) return alert("Please enter a URL");
    let key = generateKey();
    while (shortUrls[key]) {
      key = generateKey();
    }
    setShortUrls({ ...shortUrls, [key]: longUrl.trim() });
    setShortUrl(baseUrl + key);
    setLongUrl("");
  };

  useEffect(() => {
    const hashKey = window.location.hash.slice(2);
    if (hashKey && shortUrls[hashKey]) {
      const originalUrl = shortUrls[hashKey];
      setClickStats((prev) => ({
        ...prev,
        [hashKey]: (prev[hashKey] || 0) + 1,
      }));
      window.location.href = originalUrl;
    }
  }, [shortUrls]);

  return (
    <div style={{ maxWidth: 600, margin: "auto", padding: "1rem" }}>
      <h2>Client-side URL Shortener</h2>
      <input
        type="text"
        placeholder="Enter a URL"
        value={longUrl}
        onChange={(e) => setLongUrl(e.target.value)}
        style={{ width: "80%", padding: "8px" }}
      />
      <button onClick={handleShorten} style={{ padding: "8px", marginLeft: 8 }}>
        Shorten
      </button>

      {shortUrl && (
        <div style={{ marginTop: 20 }}>
          <strong>Shortened URL:</strong>{" "}
          <a href={shortUrl} target="_blank" rel="noopener noreferrer">
            {shortUrl}
          </a>
          <button
            onClick={() => {
              navigator.clipboard.writeText(shortUrl);
              alert("Copied short URL!");
            }}
            style={{ marginLeft: 10 }}
          >
            Copy
          </button>
        </div>
      )}

      <h3 style={{ marginTop: 40 }}>Stored URLs & Click Stats</h3>
      <table border="1" cellPadding="10" style={{ width: "100%" }}>
        <thead>
          <tr>
            <th>Short URL</th>
            <th>Original URL</th>
            <th>Clicks</th>
          </tr>
        </thead>
        <tbody>
          {Object.keys(shortUrls).length === 0 && (
            <tr>
              <td colSpan="3" style={{ textAlign: "center" }}>
                No URLs shortened yet.
              </td>
            </tr>
          )}
          {Object.entries(shortUrls).map(([key, url]) => (
            <tr key={key}>
              <td>
                <a href={baseUrl + key} target="_blank" rel="noopener noreferrer">
                  {baseUrl + key}
                </a>
              </td>
              <td>
                <a href={url} target="_blank" rel="noopener noreferrer">
                  {url}
                </a>
              </td>
              <td>{clickStats[key] || 0}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

export default App;
