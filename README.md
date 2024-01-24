# GeoHuntr - GeoGuessr Location Finder

*Discord: .crunny*

This script provides a tool to find the location of your current GeoGuessr game.

## Features:

- Finds and displays the location of the current GeoGuessr game.

## Usage:

Simply install the script using a userscript manager like Tampermonkey, and the tool will be available when you play GeoGuessr.

*Please note that in order to see the clickable button, once loaded into the game you must refresh the page.*

## Install

[Install GeoHuntr](https://greasyfork.org/en/scripts/485590-geohuntr)

## Script Code:

```javascript
// ==UserScript==
// @name         GeoHuntr
// @namespace    https://github.com/crunny/geohuntr/
// @version      1
// @description  A simple GeoGuessr Tool
// @author       crunny
// @match        http*://www.geoguessr.com/game/*
// @license      GPL-3.0-or-later; http://www.gnu.org/licenses/gpl-3.0.txt
// @copyright    2024+, crunny (https://github.com/crunny/)
// @homepage     https://github.com/crunny/geohuntr/
// @grant        none
// @run-at       document-start
// ==/UserScript==
// Note - You must initially refresh the page after game has loaded to enable the gui

(function () {
    let lat = 0;
    let long = 0;

    function convertCoords(lat, long) {
        const latResult = Math.abs(lat);
        const longResult = Math.abs(long);
        const dmsResult =
            Math.floor(latResult) + "°" + convertToMinutes(latResult % 1) + "'" + convertToSeconds(latResult % 1) + '"' + getLatDirection(lat) +
            "+" +
            Math.floor(longResult) + "°" + convertToMinutes(longResult % 1) + "'" + convertToSeconds(longResult % 1) + '"' + getLongDirection(long);
        return dmsResult;
    }

    function convertToMinutes(decimal) {
        return Math.floor(decimal * 60);
    }

    function convertToSeconds(decimal) {
        return (decimal * 3600 % 60).toFixed(1);
    }

    function getLatDirection(lat) {
        return lat >= 0 ? "N" : "S";
    }

    function getLongDirection(long) {
        return long >= 0 ? "E" : "W";
    }

    var originalXHR = window.XMLHttpRequest;
    var newXHR = function () {
        var xhr = new originalXHR();
        xhr.addEventListener('loadend', function () {
            if (xhr.responseURL == 'https://maps.googleapis.com/$rpc/google.internal.maps.mapsjs.v1.MapsJsInternalService/GetMetadata') {
                const respObj = JSON.parse(xhr.responseText);
                lat = respObj[1][0][5][0][1][0][2];
                long = respObj[1][0][5][0][1][0][3];
            }
        });
        return xhr;
    };
    window.XMLHttpRequest = newXHR;

    async function getCoordInfo() {
        try {
            const response = await fetch(`https://geocode.maps.co/reverse?lat=${lat}&lon=${long}`);
            if (!response.ok) {
                throw new Error('Network response was not ok');
            }
            const data = await response.json();
            return data.display_name;
        } catch (error) {
            console.error('Error:', error);
            return 'Error retrieving location information';
        }
    }

    const guiButton = document.createElement('button');
    guiButton.textContent = 'GeoHuntr';
    guiButton.style.position = 'fixed';
    guiButton.style.bottom = '220px';
    guiButton.style.left = '25px';
    guiButton.style.padding = '10px 15px';
    guiButton.style.borderRadius = '10px';
    guiButton.style.backgroundColor = 'rgba(0, 0, 0, 0.5)';
    guiButton.style.color = 'white';
    guiButton.style.fontWeight = 'bold';
    guiButton.style.fontSize = '16px';
    guiButton.style.cursor = 'help';
    guiButton.style.transition = 'background-color 0.1s';

    guiButton.addEventListener('click', function () {
        window.open(`https://www.google.be/maps/search/${convertCoords(lat, long)}`);
    });

    guiButton.addEventListener('mouseover', function () {
        guiButton.style.backgroundColor = 'rgba(0, 0, 0, 1)';
    });

    guiButton.addEventListener('mouseout', function () {
        guiButton.style.backgroundColor = 'rgba(0, 0, 0, 0.5)';
    });

    document.body.appendChild(guiButton);
})();
```

## Legal Disclaimer:

This user script, GeoHuntr, is provided on an "as is" basis without any warranty, express or implied. The author, crunny, makes no representations or warranties regarding the accuracy, completeness, or suitability of the script for any particular purpose.

The author assumes no responsibility or liability for any misuse, errors, or issues caused by the script. Users are solely responsible for the usage, implementation, and consequences of installing and running this script. 

By installing and using GeoHuntr, you acknowledge and agree that the author shall not be held responsible for any damages, losses, or adverse effects arising from the use of the script.

Use this script at your own risk.
