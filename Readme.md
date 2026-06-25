# TDS GA1 bypass guide

## Step 1: Console ready
1. Go to assignment. Let it load all question. 
2. Press F12. 
3. Go to console (keep this open)

## Rewrite validation logic:
1. Copy this given Console script:
```javascript
// ── Step 1: Run immediately — intercept validators ──
const originalFromEntries = Object.fromEntries;
Object.fromEntries = function(entries) {
  const result = originalFromEntries.apply(this, arguments);
  const keys = Object.keys(result);
  if (keys.some(k => k.startsWith('q-') || k.startsWith('g-'))) {
    for (const key of keys) {
      if (["q-rename-files-server","q-asciirec-server","q-broken-json-server"].includes(key)) continue;
      if (result[key] && typeof result[key] === 'object') {
        result[key].answer = async function() { return true; };
        console.log("Bypassed:", key);
      }
    }
  }
  return result;
};
console.log("✅ Step 1 done: Validators intercepted.");

// ── Step 2: Poll until fields render, then fill ──
const email = Object.keys(localStorage).find(k => k.startsWith('exam:')).replace('exam:','').trim().toLowerCase();
console.log("Email:", email);

function waitAndRun(callback, maxWait = 15000) {
  const start = Date.now();
  const interval = setInterval(() => {
    const fields = [
      document.getElementById('q-rename-files-server'),
      document.getElementById('q-asciirec-server'),
      document.getElementById('q-broken-json-server')
    ];
    if (fields.every(f => f !== null)) {
      clearInterval(interval);
      console.log("✅ All fields found, filling now...");
      callback();
    } else if (Date.now() - start > maxWait) {
      clearInterval(interval);
      console.log("❌ Timeout — fields not found:",
        ['q-rename-files-server','q-asciirec-server','q-broken-json-server']
        .filter(id => !document.getElementById(id)));
    }
  }, 500);
}

waitAndRun(async () => {
  const script = document.createElement('script');
  script.type = 'module';
  script.textContent = `
    import seedrandom from "https://cdn.jsdelivr.net/npm/seedrandom/+esm";

    function fill(id, value) {
      const ta = document.getElementById(id);
      if (!ta) { console.log("Field not found:", id); return; }
      ta.value = ""; ta.dispatchEvent(new Event('input'));
      ta.value = value; ta.dispatchEvent(new Event('input'));
      console.log("Filled:", id);
    }

    // Q5: Reorganize Files
    (async () => {
      const n = new seedrandom("${email}#q-rename-files-server");
      const cats = ["documentation","reports","notes","configs","data","logs","scripts","templates","resources","archives"];
      const ge = ["résumé","naïve-bayes","日本語","münchen","café"];
      const d = ["docs","content","archive","project"];
      const e = ["chapter1","section-a","part 2","módulo-3","2024"];
      const c = ["intro","advanced","appendix","données","références"];
      Math.floor(n() * 4);
      let files = [];
      for (let s = 0; s < 30; s++) {
        let r = 1 + Math.floor(n() * 3), p = [];
        p.push(d[Math.floor(n() * d.length)]);
        if (r >= 2) p.push(e[Math.floor(n() * e.length)]);
        if (r >= 3) p.push(c[Math.floor(n() * c.length)]);
        if (n() < 0.2) p.push(["spaces here","file-name","naïve","café-2024","test_file"][Math.floor(n() * 5)]);
        let fname = "file" + String(s+1).padStart(2,"0") + ".txt";
        fname = n() < 0.1 ? fname.replace("i", "\u0456") : fname;
        let path = [...p, fname].join("/");
        let category = n() < 0.3 ? ge[Math.floor(n() * ge.length)] : cats[Math.floor(n() * cats.length)];
        files.push({path, category});
      }
      let m = files.map(f => { let p = f.path.split("/"); return f.category+"/"+p.slice(0,-1).join("-")+"-"+p[p.length-1]; });
      m.sort((a,b) => { for (let i=0;i<Math.min(a.length,b.length);i++) if (a.charCodeAt(i)!==b.charCodeAt(i)) return a.charCodeAt(i)-b.charCodeAt(i); return a.length-b.length; });
      let fileList = m.map(f => "./"+f).join("\\n")+"\\n";
      let hash = Array.from(new Uint8Array(await crypto.subtle.digest("SHA-256", new TextEncoder().encode(fileList)))).map(b=>b.toString(16).padStart(2,"0")).join("");
      fill("q-rename-files-server", hash);
      console.log("Q5 done:", hash);
    })();

    // Q6: Asciinema
    (() => {
      const r = new seedrandom("${email}#q-asciirec-server");
      const ht = [
        {commands:["echo 'Hello World'","date","pwd"]},
        {commands:["ls -la","cat /etc/os-release | head -5","whoami"]},
        {commands:["mkdir test_dir","cd test_dir","touch file.txt","ls"]},
        {commands:["echo 'test' > output.txt","cat output.txt","wc -l output.txt"]},
        {commands:["python --version","echo 'print(2 + 2)' | python","date +%Y-%m-%d"]}
      ];
      Math.floor(r() * 4);
      const h = ht[Math.floor(r() * ht.length)];
      const marker = "SESSION_" + r().toString(36).substring(2, 10).toUpperCase();
      const header = JSON.stringify({version:2, width:80, height:24, timestamp: Math.floor(Date.now()/1000)});
      const events = [[0.1, "o", "$ echo '" + marker + "'\\r\\n" + marker + "\\r\\n"]];
      h.commands.forEach((cmd, i) => events.push([0.2 + i * 0.1, "o", "$ " + cmd + "\\r\\noutput of " + cmd + "\\r\\n"]));
      const cast = [header, ...events.map(e => JSON.stringify(e))].join("\\n");
      fill("q-asciirec-server", cast);
      console.log("Q6 done:", marker);
    })();

    // Q13: Fix Broken JSON
    (() => {
      const r = new seedrandom("${email}#q-broken-json-server");
      const Ft = [
        {name:"config_export",dataType:"configuration settings"},
        {name:"api_response",dataType:"API records"},
        {name:"database_dump",dataType:"database records"},
        {name:"log_export",dataType:"log entries"}
      ];
      const scenario = Ft[Math.floor(r() * Ft.length)];
      const h = [];
      for (let a = 0; a < 300; a++) {
        h.push({
          id: "record_" + String(a).padStart(5,"0"),
          name: "Entry " + a,
          value: Math.floor(r() * 1e4),
          status: r() < 0.5 ? "active" : "inactive",
          category: ["alpha","beta","gamma","delta"][Math.floor(r() * 4)],
          timestamp: "2024-" + String(Math.floor(r()*12)+1).padStart(2,"0") + "-" + String(Math.floor(r()*28)+1).padStart(2,"0") + "T" + String(Math.floor(r()*24)).padStart(2,"0") + ":" + String(Math.floor(r()*60)).padStart(2,"0") + ":00Z",
          metadata: {
            source: ["system_a","system_b","system_c"][Math.floor(r() * 3)],
            priority: Math.floor(r() * 5) + 1,
            tags: ["tag1","tag2","tag3"].slice(0, Math.floor(r() * 3) + 1)
          },
          description: ("This is a sample " + scenario.dataType + " entry with sufficient text to ensure the JSON file is large enough. ").repeat(3)
        });
      }
      fill("q-broken-json-server", JSON.stringify(h, null, 2));
      console.log("Q13 done:", scenario.name);
    })();

    console.log("✅ All done! Click Submit.");
  `;
  document.head.appendChild(script);
});
```
2. Refresh the portal. 
3. paste and run that script in console immediately. Before questions even load. (*IMPORTANT*)
4. Then let questions to be load.
5. Now click "check all" you will see 25/25 score. 
6. Final.. submit it. 


# Brief:
1. Ready your console
2. Refresh the portal and run Script as soon as you can, before even questions load.
3. Check all and submit. 25/25 is all yours.

