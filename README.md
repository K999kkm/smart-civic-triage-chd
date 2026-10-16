# 🏙️ Smart Civic Triage - Chandigarh

An AI-powered Digital Public Infrastructure (DPI) platform designed to modernize civic issue reporting and municipal response in Chandigarh. Built for the HackCulture Hackathon.

## 🚀 The Problem
Traditional civic reporting relies on manual data entry, human verification, and reactive planning. This creates bottlenecks, language barriers for citizens, and inefficient routing for Municipal Corporations.

## 💡 The Solution
Smart Civic Triage bridges the gap between citizens and city planners using multimodal AI. Citizens snap a photo of an issue, and the AI instantly categorizes the defect, calculates severity, and routes it to the appropriate municipal sector. 

## ✨ Key Features
*   **🤖 AI Vision Triage:** Powered by Google Gemini, the app acts as an expert civil engineer. It instantly analyzes photos to identify specific infrastructure defects (e.g., Potholes, Waterlogging, Alligator Cracking, Raveling) and assigns a severity score (1-10).
*   **🌍 DPI Multi-Language Accessibility:** True Digital Public Infrastructure must be accessible to all. The UI and AI outputs dynamically translate between English, Hindi (हिंदी), and Punjabi (ਪੰਜਾਬੀ) with a single click.
*   **📍 Uber-Style Geolocation:** Integrated Leaflet mapping allows citizens to easily drop a pin on their exact location for highly accurate reporting.
*   **🚨 Incident Command Dashboard:** A dedicated portal for municipal authorities featuring live ticket tracking and an Esri-powered **AI Predictive Heatmap** to shift city maintenance from reactive to proactive.

## 🛠️ Tech Stack
*   **Frontend:** HTML5, Tailwind CSS, Vanilla JavaScript
*   **Mapping:** Leaflet.js, Esri Dark Gray Canvas, OpenStreetMap Nominatim API
*   **AI Engine:** Google Gemini (Multimodal Vision API)

## 📱 How to Run
This project is built as a lightweight, client-side web application.
1. Clone the repository or download `index.html`.
2. Open `index.html` in any modern web browser.
3. *(Optional for live AI)*: Insert your Google Gemini API key into the `GEMINI_API_KEY` variable in the script.