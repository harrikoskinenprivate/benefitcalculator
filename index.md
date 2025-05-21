<!DOCTYPE html>
<html lang="fi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="icon" href="data:,">
    <title>Hyötylaskuri</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-image: url('background.svg');
            background-size: cover;
            background-attachment: fixed;
            margin: 0;
            padding: 20px;
            color: #333;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
        }
        .input-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input {
            width: 200px;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        button {
            background-color: #005b9f;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        .results-container {
            display: flex;
            gap: 20px;
            margin-top: 30px;
        }
        .results {
            flex: 1;
            padding: 20px;
            background: #f5f5f5;
            border-radius: 4px;
        }
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
        }
        .modal-content {
            background: white;
            margin: 5% auto;
            padding: 20px;
            width: 90%;
            max-width: 800px;
            border-radius: 8px;
            max-height: 85vh;
            display: flex;
            flex-direction: column;
        }
        .modal-content h2 {
            font-size: 1.5em;
            margin-bottom: 15px;
        }
        .modal-content h3 {
            font-size: 1.2em;
            margin-bottom: 10px;
        }
        .modal-content p {
            font-size: 0.9em;
            margin: 5px 0;
        }
        .modal-content button {
            margin-top: 15px;
            align-self: center;
        }
        #erittely {
            overflow-y: auto;
            max-height: calc(85vh - 150px);
            padding-right: 10px;
        }
        .difference {
            margin-top: 20px;
            padding: 15px;
            background-color: #ffebee;
            border: 1px solid #ef5350;
            border-radius: 4px;
            color: #c62828;
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
        }
        .difference h3 {
            margin: 0;
            color: #b71c1c;
            text-align: center;
        }
        .difference-row {
            display: flex;
            justify-content: center;
            gap: 40px;
            margin-top: 10px;
        }
        @media screen and (max-width: 768px) {
            .results-container {
                flex-direction: column;
                gap: 10px;
            }
            .difference-row {
                flex-direction: column;
                gap: 10px;
                align-items: center;
                text-align: center;
            }
            .container {
                padding: 15px;
            }
            .results {
                padding: 15px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hyötylaskuri</h1>

        <div class="input-group">
            <label>Toimeksiantojen määrä (kpl/kk)</label>
            <input type="number" id="toimeksiannot" value="11">
        </div>

        <div class="input-group">
            <label>Henkilöitä per toimeksianto</label>
            <input type="number" id="hloa_per_toimeksianto" value="12">
        </div>

        <div class="input-group">
            <label>Tuntihinta (€)</label>
            <input type="number" id="tuntihinta" value="35">
        </div>

        <button onclick="laskeTulokset()">Laske</button>

        <div class="results-container">
            <div class="results">
                <h2>Nykyinen toimintamalli</h2>
                <p>Käsittelyaika: <span id="tunnit_a">0</span> h/kk</p>
                <p>Kustannukset: <span id="kustannus_a">0</span> €/kk</p>
            </div>

            <div class="results">
                <h2>Turvallisuusportaali käytössä</h2>
                <p>Käsittelyaika: <span id="tunnit_b">0</span> h/kk</p>
                <p>Kustannukset: <span id="kustannus_b">0</span> €/kk</p>
            </div>
        </div>

        <div style="text-align: center; margin: 20px 0;">
            <button onclick="naytaErittely()">Näytä käsittelyajan erittely</button>
        </div>

        <div class="difference">
            <h3>Erotus (Turvallisuusportaali - Nykyinen malli)</h3>
            <div class="difference-row">
                <p>Käsittelyajan erotus: <span id="tunnit_erotus">0</span> h/kk</p>
                <p>Kustannusten erotus: <span id="kustannus_erotus">0</span> €/kk</p>
            </div>
        </div>
    </div>

    <div id="erittelyModal" class="modal">
        <div class="modal-content">
            <h2 id="erittelyOtsikko">Käsittelyajan erittely</h2>
            <div id="erittely"></div>
            <button onclick="suljeErittely()">Sulje</button>
        </div>
    </div>

    <script>
        let erittelyDataA = [];
        let erittelyDataB = [];
        let dataA = null;
        let dataB = null;

        // Ladataan JSON-tiedostot
        async function lataaData() {
            try {
                const responseA = await fetch('a.json');
                const responseB = await fetch('b.json');
                dataA = await responseA.json();
                dataB = await responseB.json();
                console.log('Ladattu data A:', dataA);
                console.log('Ladattu data B:', dataB);
                laskeTulokset();
            } catch (error) {
                console.error('Virhe datan lataamisessa:', error);
            }
        }

        function laskeTiedostosta(data, toimeksiannot, hloa_per_toimeksianto, tuntihinta) {
            const henkilomaara = toimeksiannot * hloa_per_toimeksianto;
            let kokonaisminuutit = 0;
            const erittely = [];

            for (const tehtava of Object.values(data.kasittelyajat)) {
                const tehtava_aika = (
                    tehtava.aika *
                    henkilomaara *
                    (tehtava.osuus / 100)
                );
                kokonaisminuutit += tehtava_aika;

                erittely.push({
                    nimi: tehtava.nimi,
                    aika: Math.round(tehtava_aika / 60 * 100) / 100,
                    min_per_hlo: tehtava.aika,
                    osuus_prosentteina: tehtava.osuus
                });
            }

            const tunnit = kokonaisminuutit / 60;
            const kustannus = tunnit * tuntihinta;

            return {
                tunnit: Math.round(tunnit * 100) / 100,
                kustannus: Math.round(kustannus * 100) / 100,
                erittely: erittely
            };
        }

        function laskeTulokset() {
            if (!dataA || !dataB) return;

            const toimeksiannot = parseFloat(document.getElementById('toimeksiannot').value);
            const hloa_per_toimeksianto = parseFloat(document.getElementById('hloa_per_toimeksianto').value);
            const tuntihinta = parseFloat(document.getElementById('tuntihinta').value);

            const tulos_a = laskeTiedostosta(dataA, toimeksiannot, hloa_per_toimeksianto, tuntihinta);
            const tulos_b = laskeTiedostosta(dataB, toimeksiannot, hloa_per_toimeksianto, tuntihinta);

            // Päivitä A tulokset
            document.getElementById('tunnit_a').textContent = tulos_a.tunnit;
            document.getElementById('kustannus_a').textContent = tulos_a.kustannus;
            erittelyDataA = tulos_a.erittely;

            // Päivitä B tulokset
            document.getElementById('tunnit_b').textContent = tulos_b.tunnit;
            document.getElementById('kustannus_b').textContent = tulos_b.kustannus;
            erittelyDataB = tulos_b.erittely;

            // Laske ja päivitä erotukset
            const tunnit_erotus = (tulos_b.tunnit - tulos_a.tunnit).toFixed(2);
            const kustannus_erotus = (tulos_b.kustannus - tulos_a.kustannus).toFixed(2);

            document.getElementById('tunnit_erotus').textContent = tunnit_erotus;
            document.getElementById('kustannus_erotus').textContent = kustannus_erotus;
        }

        function naytaErittely() {
            const erittelyHtml = `
                <div style="display: flex; gap: 20px; margin-bottom: 10px; padding: 5px; border-bottom: 1px solid #eee;">
                    <div style="flex: 1;">
                        <strong>Tehtävä</strong>
                    </div>
                    <div style="flex: 1; text-align: center;">
                        <small style="font-size: 0.8em; color: #666;">Nykyinen toimintamalli</br>Kokonaisaika(osuus)</small>
                    </div>
                    <div style="flex: 1; text-align: center;">
                        <small style="font-size: 0.8em; color: #666;">Turvallisuusportaali käytössä</br>Kokonaisaika(osuus)</small>
                    </div>
                </div>
                ${erittelyDataA.map((item, index) => {
                    const itemB = erittelyDataB[index];
                    return `
                        <div style="display: flex; gap: 20px; margin-bottom: 10px; padding: 5px; border-bottom: 1px solid #eee;">
                            <div style="flex: 1;">
                                <strong>${item.nimi}</strong>
                            </div>
                            <div style="flex: 1; text-align: center;">
                                ${item.aika} h (${item.osuus_prosentteina}%)<br>
                                <small>${item.min_per_hlo} min/hlö</small>
                            </div>
                            <div style="flex: 1; text-align: center;">
                                ${itemB.aika} h (${itemB.osuus_prosentteina}%)<br>
                                <small>${itemB.min_per_hlo} min/hlö</small>
                            </div>
                        </div>
                    `;
                }).join('')}
            `;

            document.getElementById('erittely').innerHTML = erittelyHtml;
            document.getElementById('erittelyModal').style.display = 'block';
        }

        function suljeErittely() {
            document.getElementById('erittelyModal').style.display = 'none';
        }

        // Ladataan data ja lasketaan tulokset heti sivun latauduttua
        lataaData();
    </script>
</body>
</html>
