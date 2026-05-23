"use client"

import { useEffect, useState } from "react"
import axios from "axios"
import { motion } from "framer-motion"

export default function Home() {
  const [weather, setWeather] = useState(null)
  const [forecast, setForecast] = useState([])
  const [alerts, setAlerts] = useState([])
  const [updated, setUpdated] = useState("")
  const [loading, setLoading] = useState(true)

  async function fetchWeather() {
    try {
      // Current Conditions
      const currentObs = await axios.get(
        "https://api.weather.gov/stations/KILG/observations/latest"
      )

      // Forecast
      const forecastData = await axios.get(
        "https://api.weather.gov/gridpoints/PHI/49,75/forecast"
      )

      // Delaware Alerts
      const alertData = await axios.get(
        "https://api.weather.gov/alerts/active?area=DE"
      )

      setWeather(currentObs.data.properties)
      setForecast(forecastData.data.properties.periods)
      setAlerts(alertData.data.features)
      setUpdated(new Date().toLocaleTimeString())
      setLoading(false)
    } catch (err) {
      console.error("Weather Fetch Error:", err)
    }
  }

  // AUTO REFRESH EVERY 5 MINUTES
  useEffect(() => {
    fetchWeather()

    const interval = setInterval(() => {
      fetchWeather()
    }, 300000)

    return () => clearInterval(interval)
  }, [])

  if (loading) {
    return (
      <div className="min-h-screen bg-[#07111f] flex items-center justify-center text-white text-4xl font-black">
        Loading DelawareWx...
      </div>
    )
  }

  return (
    <main className="min-h-screen bg-[#07111f] text-white">
      {/* HEADER */}
      <header className="sticky top-0 z-50 bg-[#08192d]/95 backdrop-blur-xl border-b border-cyan-900/30">
        <div className="max-w-7xl mx-auto px-6 py-5 flex flex-col lg:flex-row lg:items-center lg:justify-between gap-4">
          <div>
            <h1 className="text-5xl font-black text-cyan-300 tracking-tight">
              DelawareWx
            </h1>

            <p className="text-slate-400 mt-2">
              Delaware Weather Intelligence Platform
            </p>
          </div>

          <div className="text-right">
            <div className="bg-emerald-500/20 border border-emerald-400/30 px-4 py-2 rounded-full inline-block text-emerald-300 font-bold text-sm">
              LIVE DATA STREAM
            </div>

            <div className="text-slate-400 mt-3 text-sm">
              Updated {updated}
            </div>
          </div>
        </div>
      </header>

      {/* HERO */}
      <section className="max-w-7xl mx-auto px-6 py-8">
        <div className="bg-gradient-to-r from-cyan-500/20 to-blue-500/10 border border-cyan-400/20 rounded-3xl p-8 shadow-2xl">
          <div className="grid lg:grid-cols-2 gap-10 items-center">
            <div>
              <div className="text-sm text-cyan-300 font-bold uppercase tracking-widest mb-3">
                Operational Forecast Center
              </div>

              <h2 className="text-5xl font-black leading-tight mb-5">
                Delaware Weather.
                <br />
                Live. Modern. Operational.
              </h2>

              <p className="text-slate-300 leading-relaxed text-lg">
                Fully operational Delaware weather intelligence platform using
                NOAA, radar integrations, live alerts, and automatic updates.
              </p>

              <div className="flex gap-4 mt-8 flex-wrap">
                <button className="bg-cyan-400 hover:bg-cyan-300 transition px-6 py-4 rounded-2xl text-slate-950 font-black shadow-lg">
                  Open Radar
                </button>

                <button className="bg-slate-800 hover:bg-slate-700 transition px-6 py-4 rounded-2xl font-bold border border-slate-700">
                  Forecast Center
                </button>
              </div>
            </div>

            <div className="relative rounded-3xl overflow-hidden border border-cyan-400/20 h-[350px]">
              <img
                src="https://images.unsplash.com/photo-1504608524841-42fe6f032b4b?q=80&w=1600&auto=format&fit=crop"
                alt="Radar"
                className="w-full h-full object-cover opacity-70"
              />

              <div className="absolute inset-0 bg-gradient-to-t from-[#07111f] via-transparent to-transparent" />

              <div className="absolute bottom-6 left-6">
                <div className="text-6xl font-black">
                  {Math.round(weather.temperature.value * 9 / 5 + 32)}°
                </div>

                <div className="text-cyan-300 text-xl mt-2">
                  {weather.textDescription}
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      {/* MAIN GRID */}
      <section className="max-w-7xl mx-auto px-6 pb-8 grid lg:grid-cols-3 gap-6">
        {/* CURRENT CONDITIONS */}
        <motion.div
          initial={{ opacity: 0, y: 15 }}
          animate={{ opacity: 1, y: 0 }}
          className="bg-[#102540] rounded-3xl p-6 border border-cyan-900/20 shadow-2xl"
        >
          <h2 className="text-2xl font-black text-cyan-300 mb-6">
            Current Conditions
          </h2>

          <div className="grid grid-cols-2 gap-4">
            <ConditionCard
              title="Temperature"
              value={`${Math.round(
                weather.temperature.value * 9 / 5 + 32
              )}°`}
            />

            <ConditionCard
              title="Humidity"
              value={`${Math.round(
                weather.relativeHumidity.value
              )}%`}
            />

            <ConditionCard
              title="Dewpoint"
              value={`${Math.round(
                weather.dewpoint.value * 9 / 5 + 32
              )}°`}
            />

            <ConditionCard
              title="Wind"
              value={`${
                weather.windSpeed.value
                  ? Math.round(weather.windSpeed.value / 1.609)
                  : 0
              } mph`}
            />

            <ConditionCard
              title="Pressure"
              value={`${Math.round(
                weather.barometricPressure.value / 100
              )} mb`}
            />

            <ConditionCard
              title="Visibility"
              value={`${
                weather.visibility.value
                  ? Math.round(weather.visibility.value / 1609)
                  : 10
              } mi`}
            />
          </div>
        </motion.div>

        {/* FORECAST */}
        <motion.div
          initial={{ opacity: 0, y: 15 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.1 }}
          className="bg-[#102540] rounded-3xl p-6 border border-cyan-900/20 shadow-2xl"
        >
          <h2 className="text-2xl font-black text-cyan-300 mb-6">
            Delaware Forecast
          </h2>

          <div className="space-y-4">
            {forecast.slice(0, 5).map((period, index) => (
              <div
                key={index}
                className="bg-[#142f52] rounded-2xl p-5 border border-cyan-900/10"
              >
                <div className="flex justify-between items-center mb-3">
                  <div className="font-black text-cyan-300">
                    {period.name}
                  </div>

                  <div className="text-xl font-black">
                    {period.temperature}°
                    {period.temperatureUnit}
                  </div>
                </div>

                <div className="text-slate-300 text-sm leading-relaxed">
                  {period.shortForecast}
                </div>
              </div>
            ))}
          </div>
        </motion.div>

        {/* ALERTS */}
        <motion.div
          initial={{ opacity: 0, y: 15 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.2 }}
          className="bg-[#102540] rounded-3xl p-6 border border-cyan-900/20 shadow-2xl"
        >
          <h2 className="text-2xl font-black text-cyan-300 mb-6">
            Active Alerts
          </h2>

          <div className="space-y-4">
            {alerts.length === 0 && (
              <div className="bg-emerald-500/20 border border-emerald-400/20 rounded-2xl p-5 text-emerald-300 font-bold">
                No active Delaware alerts.
              </div>
            )}

            {alerts.slice(0, 5).map((alert, index) => (
              <div
                key={index}
                className="bg-amber-500/10 border border-amber-400/20 rounded-2xl p-5"
              >
                <div className="text-amber-300 font-black text-lg mb-3">
                  {alert.properties.event}
                </div>

                <div className="text-slate-300 text-sm leading-relaxed">
                  {alert.properties.headline}
                </div>
              </div>
            ))}
          </div>
        </motion.div>
      </section>

      {/* RADAR */}
      <section className="max-w-7xl mx-auto px-6 pb-10">
        <div className="bg-[#102540] rounded-3xl overflow-hidden border border-cyan-900/20 shadow-2xl">
          <div className="p-6 border-b border-cyan-900/20 flex justify-between items-center">
            <div>
              <h2 className="text-3xl font-black text-cyan-300">
                Radar Center
              </h2>

              <p className="text-slate-400 mt-2">
                RainViewer Live Radar Integration
              </p>
            </div>

            <div className="bg-red-500/20 border border-red-400/20 px-4 py-2 rounded-full text-red-300 text-sm font-bold">
              LIVE RADAR
            </div>
          </div>

          <iframe
            src="https://www.rainviewer.com/map.html"
            className="w-full h-[700px]"
          />
        </div>
      </section>

      {/* FOOTER */}
      <footer className="border-t border-cyan-900/20 py-10 mt-10">
        <div className="max-w-7xl mx-auto px-6 flex flex-col lg:flex-row justify-between gap-6">
          <div>
            <div className="text-3xl font-black text-cyan-300">
              DelawareWx
            </div>

            <div className="text-slate-400 mt-2">
              Data Over Drama ™
            </div>
          </div>

          <div className="text-slate-500 text-sm">
            Powered by NOAA, RainViewer, Open APIs, and modern web
            infrastructure.
          </div>
        </div>
      </footer>
    </main>
  )
}

function ConditionCard({ title, value }) {
  return (
    <div className="bg-[#142f52] rounded-2xl p-5 border border-cyan-900/10">
      <div className="text-slate-400 text-sm mb-3">
        {title}
      </div>

      <div className="text-3xl font-black text-cyan-300">
        {value}
      </div>
    </div>
  )
}
