

require "import"
import "android.widget.*"
import "com.androlua.*"
import "android.view.View"
import "java.io.File"
import "android.content.Intent"
import "android.net.Uri"
import "cjson"
import "com.androlua.LuaDialog"
import "android.view.*"
import "android.content.Context"
import "android.view.accessibility.AccessibilityEvent" 
import "android.os.Handler"
import "android.os.Looper"
import "android.widget.AdapterView" 
import "android.R" -- Diperlukan untuk layout spinner

local vibrator = this.getSystemService(Context.VIBRATOR_SERVICE)
local context = this 
local mainHandler = Handler(Looper.getMainLooper())

-- =======================================
-- API URL
-- =======================================
local API_GIST = "https://zelapioffciall.koyeb.app/tools/gist?url="
local API_PASTEBIN = "https://zelapioffciall.koyeb.app/tools/pastebin?url="
local API_RUTE = "https://zelapioffciall.koyeb.app/tools/rute?" -- BARU

local API_APKPURE = "https://api.siputzx.my.id/api/apk/apkpure"

local API_AN1 = "https://api.siputzx.my.id/api/apk/an1"

local API_APKMODY = "https://api.siputzx.my.id/api/apk/apkmody"

local API_BERITA = "https://infoqueries.itz-ashlynn.workers.dev/"

-- =======================================
-- FUNGSI GLOBAL: AKSESIBILITAS
-- =======================================
local function announceForAccessibility(text)
    if not text then return end 
    local accessibilityManager = context.getSystemService(Context.ACCESSIBILITY_SERVICE)
    if accessibilityManager and accessibilityManager.isEnabled() and accessibilityManager.isTouchExplorationEnabled() then
        local announcement = AccessibilityEvent.obtain(AccessibilityEvent.TYPE_ANNOUNCEMENT)
        announcement.setPackageName(context.getPackageName())
        announcement.setClassName(context.getClass().getName())
        announcement.getText().add(text)
        accessibilityManager.sendAccessibilityEvent(announcement)
    end
end

-- =======================================
-- LAYOUT UTAMA (DENGAN SPINNER)
-- =======================================
layout = {
  LinearLayout,
  orientation = "vertical",
  layout_width = "fill",
  layout_height = "fill",
  padding = "10dp",
  {
    TextView,
    text = "馃Z" .. " Pusat Alat " .. "馃Z",
    textSize = "20sp",
    gravity = "center",
    textColor = "#FFFFFF",
    layout_marginBottom = "10dp",
  },
  {
    Spinner,
    id = "toolSpinner",
    layout_width = "fill",
    layout_height = "wrap_content",
  },
  {
    FrameLayout, -- Kontainer untuk menampung alat
    layout_width = "fill",
    layout_height = "fill",
    layout_weight = "1", 
    layout_marginTop = "10dp",
    {
      -- ==================================
      -- Layout 1: Gist Downloader
      -- ==================================
      LinearLayout,
      id = "gist_layout",
      orientation = "vertical",
      layout_width = "fill",
      layout_height = "fill",
      visibility = View.GONE,
      {
        TextView,
        text = "Masukkan URL Gist:",
        textSize = "14sp",
      },
      {
        EditText,
        id = "gist_urlInput",
        hint = "Contoh: https://gist.github.com/...",
        singleLine = true,
        layout_width = "fill",
        layout_height = "wrap_content",
      },
      {
        Button,
        id = "gist_fetchButton",
        text = "Ambil Data Gist",
        layout_marginTop = "5dp",
      },
      {
        ScrollView, 
        layout_width = "fill",
        layout_height = "fill",
        layout_weight = "1",
        layout_marginTop = "10dp",
        {
          TextView,
          id = "gist_hasilText",
          text = "Hasil akan muncul di sini...",
          textSize = "12sp",
          textColor = "#DDDDDD",
        }
      }
    },

{
	LinearLayout,
	id = "apk_layout",
	orientation = "vertical",
	layout_width = "fill",
	layout_height = "fill",
	visibility = View.GONE,
	{
		TextView,
		text = "Cari APK di APKPure:",
		textSize = "14sp",
		},
	{
		EditText,
		id = "apk_searchInput",
		hint = "Ketik nama APK (cth: Mobile Legends)",
		singleLine = true,
		layout_width = "fill",
		layout_height = "wrap_content",
		},
	{
		Button,
		id = "apk_fetchButton",
		text = "Cari APK",
		layout_marginTop = "5dp",
		},
	{
		TextView,
		id = "apk_statusText",
		text = "Masukkan nama APK lalu tekan 'Cari APK'.",
		layout_marginTop="5dp",
		textColor="#FFFF00",
		},
	{
		ScrollView,
		layout_width = "fill",
		layout_height = "fill",
		layout_weight = "1",
		layout_marginTop = "10dp",
		{
			LinearLayout,
			id = "apk_result_container",
			orientation = "vertical",
			layout_width = "fill",
			layout_height = "wrap_content",
			}
		}
	},

{
	LinearLayout,
	id = "an1_layout",
	orientation = "vertical",
	layout_width = "fill",
	layout_height = "fill",
	visibility = View.GONE,
	{
		TextView,
		text = "Cari APK di AN1 (MODs):",
		textSize = "14sp",
		},
	{
		EditText,
		id = "an1_searchInput",
		hint = "Ketik nama APK (cth: Pou)",
		singleLine = true,
		layout_width = "fill",
		layout_height = "wrap_content",
		},
	{
		Button,
		id = "an1_fetchButton",
		text = "Cari di AN1",
		layout_marginTop = "5dp",
		},
	{
		TextView,
		id = "an1_statusText",
		text = "Masukkan nama APK lalu tekan 'Cari di AN1'.",
		layout_marginTop="5dp",
		textColor="#FFFF00",
		},
	{
		ScrollView,
		layout_width = "fill",
		layout_height = "fill",
		layout_weight = "1",
		layout_marginTop = "10dp",
		{
			LinearLayout,
			id = "an1_result_container",
			orientation = "vertical",
			layout_width = "fill",
			layout_height = "wrap_content",
			}
		}
	},

{
	LinearLayout,
	id = "apkmody_layout",
	orientation = "vertical",
	layout_width = "fill",
	layout_height = "fill",
	visibility = View.GONE,
	{
		TextView,
		text = "Cari APK di APKMody:",
		textSize = "14sp",
		},
	{
		EditText,
		id = "apkmody_searchInput",
		hint = "Ketik nama APK (cth: free fire)",
		singleLine = true,
		layout_width = "fill",
		layout_height = "wrap_content",
		},
	{
		Button,
		id = "apkmody_fetchButton",
		text = "Cari di APKMody",
		layout_marginTop = "5dp",
		},
	{
		TextView,
		id = "apkmody_statusText",
		text = "Masukkan nama APK lalu tekan 'Cari di APKMody'.",
		layout_marginTop="5dp",
		textColor="#FFFF00",
		},
	{
		ScrollView,
		layout_width = "fill",
		layout_height = "fill",
		layout_weight = "1",
		layout_marginTop = "10dp",
		{
			LinearLayout,
			id = "apkmody_result_container",
			orientation = "vertical",
			layout_width = "fill",
			layout_height = "wrap_content",
			}
		}
	},

{
	LinearLayout,
	id = "berita_layout",
	orientation = "vertical",
	layout_width = "fill",
	layout_height = "fill",
	visibility = View.GONE,
	{
		TextView,
		text = "Cari Berita (Web):",
		textSize = "14sp",
		},
	{
		EditText,
		id = "berita_searchInput",
		hint = "Ketik kueri pencarian...",
		singleLine = true,
		layout_width = "fill",
		layout_height = "wrap_content",
		},
	{
		Button,
		id = "berita_fetchButton",
		text = "Cari Berita",
		layout_marginTop = "5dp",
		},
	{
		TextView,
		id = "berita_statusText",
		text = "Masukkan kueri lalu tekan 'Cari Berita'.",
		layout_marginTop="5dp",
		textColor="#FFFF00",
		},
	{
		ScrollView,
		layout_width = "fill",
		layout_height = "fill",
		layout_weight = "1",
		layout_marginTop = "10dp",
		{
			LinearLayout,
			id = "berita_result_container",
			orientation = "vertical",
			layout_width = "fill",
			layout_height = "wrap_content",
			}
		}
	},

    {
      -- ==================================
      -- Layout 2: Pastebin Downloader
      -- ==================================
      LinearLayout,
      id = "pb_layout",
      orientation = "vertical",
      layout_width = "fill",
      layout_height = "fill",
      visibility = View.GONE,
      {
        TextView,
        text = "Masukkan URL Pastebin:",
        textSize = "14sp",
      },
      {
        EditText,
        id = "pb_urlInput",
        hint = "Contoh: https://pastebin.com/...",
        singleLine = true,
        layout_width = "fill",
        layout_height = "wrap_content",
      },
      {
        Button,
        id = "pb_fetchButton",
        text = "Ambil Data Pastebin",
        layout_marginTop = "5dp",
      },
      {
        ScrollView, 
        layout_width = "fill",
        layout_height = "fill",
        layout_weight = "1",
        layout_marginTop = "10dp",
        {
          TextView,
          id = "pb_hasilText",
          text = "Hasil akan muncul di sini...",
          textSize = "12sp",
          textColor = "#DDDDDD",
        }
      }
    },
    {
      -- ==================================
      -- Layout 3: Cari Rute (BARU)
      -- ==================================
      LinearLayout,
      id = "rute_layout",
      orientation = "vertical",
      layout_width = "fill",
      layout_height = "fill",
      visibility = View.GONE,
      {
        LinearLayout, -- Kontainer horizontal untuk input
        orientation = "horizontal",
        layout_width = "fill",
        layout_height = "wrap_content",
        {
          EditText,
          id = "rute_fromInput",
          hint = "Dari (Asal)",
          singleLine = true,
          layout_width = "0dp",
          layout_weight = "1",
          layout_height = "wrap_content",
        },
        {
          EditText,
          id = "rute_toInput",
          hint = "Ke (Tujuan)",
          singleLine = true,
          layout_width = "0dp",
          layout_weight = "1",
          layout_height = "wrap_content",
          layout_marginLeft = "5dp",
        },
      },
      {
        Button,
        id = "rute_fetchButton",
        text = "Cari Rute",
        layout_marginTop = "5dp",
      },
      {
        ScrollView, 
        layout_width = "fill",
        layout_height = "fill",
        layout_weight = "1",
        layout_marginTop = "10dp",
        {
          TextView,
          id = "rute_hasilText",
          text = "Hasil akan muncul di sini...",
          textSize = "12sp",
          textColor = "#DDDDDD",
        }
      }
    },
    --[[
      -- ==================================
      -- Layout 4: ALAT BERIKUTNYA
      -- ==================================
    --]]
  },
  {
    Button,
    id = "exitButton",
    text = "Keluar",
    layout_width = "wrap_content",
    layout_height = "wrap_content",
    layout_gravity = "center_horizontal",
    layout_marginTop = "10dp",
  }
}

dlg = LuaDialog(this)
dlg.setTitle("Pusat Alat v3.0") -- Versi naik
dlg.setView(loadlayout(layout))

-- =======================================
-- =======================================
-- LOGIKA 1: GIST DOWNLOADER
-- =======================================
-- =======================================
function fetchGistData()
  local gistUrl = gist_urlInput.text
  if gistUrl == "" then
    gist_hasilText.text = "URL tidak boleh kosong!"
    announceForAccessibility(gist_hasilText.text)
    return
  end
  
  gist_hasilText.text = "Mengambil data Gist..."
  announceForAccessibility(gist_hasilText.text)
  vibrator.vibrate(50)
  
  local fullApiUrl = API_GIST .. gistUrl
  
  Http.get(fullApiUrl, function(code, content)
    if code == 200 then
      local success, data = pcall(cjson.decode, content)
      
      if success and data and data.status == true and data.files then
        local formattedResult = "Berhasil Ditemukan:\n\n"
        formattedResult = formattedResult .. "ID Gist: " .. (data.gist_id or "Tidak ada") .. "\n"
        formattedResult = formattedResult .. "Creator: " .. (data.creator or "Tidak ada") .. "\n\n"
        formattedResult = formattedResult .. "--- DAFTAR FILE --- \n\n"
        
        local fileCount = 0
        for filename, filecontent in pairs(data.files) do
          fileCount = fileCount + 1
          formattedResult = formattedResult .. "File: " .. filename .. "\n"
          formattedResult = formattedResult .. "Konten:\n"
          formattedResult = formattedResult .. "--------------------\n"
          formattedResult = formattedResult .. (filecontent or "[Konten kosong]") .. "\n"
          formattedResult = formattedResult .. "--------------------\n\n"
        end
        
        if fileCount == 0 then
          formattedResult = "Data Gist ditemukan, tapi tidak ada file di dalamnya."
        end
        
        gist_hasilText.text = formattedResult
        announceForAccessibility("Data Gist berhasil dimuat.")
        
      elseif success and data and data.status == false then
         gist_hasilText.text = "Gagal mengambil data: " .. (data.message or "Server mengembalikan status false.")
         announceForAccessibility(gist_hasilText.text)
      else
        gist_hasilText.text = "Gagal memproses data JSON yang diterima."
        announceForAccessibility(gist_hasilText.text)
      end
    else
      gist_hasilText.text = "Gagal terhubung ke server. Kode: " .. code
      announceForAccessibility(gist_hasilText.text)
    end
  end)
end
gist_fetchButton.onClick = function()
  fetchGistData()
end
function resetGistDownloader()
  gist_urlInput.setText("")
  gist_hasilText.text = "Masukkan URL Gist dan tekan tombol ambil data."
end


-- =======================================
-- =======================================
-- LOGIKA 2: PASTEBIN DOWNLOADER
-- =======================================
-- =======================================
function fetchPastebinData()
  local pbUrl = pb_urlInput.text
  if pbUrl == "" then
    pb_hasilText.text = "URL tidak boleh kosong!"
    announceForAccessibility(pb_hasilText.text)
    return
  end
  
  pb_hasilText.text = "Mengambil data Pastebin..."
  announceForAccessibility(pb_hasilText.text)
  vibrator.vibrate(50)
  
  local fullApiUrl = API_PASTEBIN .. pbUrl
  
  Http.get(fullApiUrl, function(code, content)
    if code == 200 then
      local success, data = pcall(cjson.decode, content)
      
      if success and data and data.status == true and data.content then
        local formattedResult = "Berhasil Ditemukan:\n\n"
        formattedResult = formattedResult .. "ID Paste: " .. (data.paste_id or "Tidak ada") .. "\n"
        formattedResult = formattedResult .. "Creator: " .. (data.creator or "Tidak ada") .. "\n\n"
        formattedResult = formattedResult .. "--- KONTEN --- \n\n"
        formattedResult = formattedResult .. (data.content or "[Konten kosong]")
        
        pb_hasilText.text = formattedResult
        announceForAccessibility("Data Pastebin berhasil dimuat.")
        
      elseif success and data and data.status == false then
         pb_hasilText.text = "Gagal mengambil data: " .. (data.message or "Server mengembalikan status false.")
         announceForAccessibility(pb_hasilText.text)
      else
        pb_hasilText.text = "Gagal memproses data JSON yang diterima."
        announceForAccessibility(pb_hasilText.text)
      end
    else
      pb_hasilText.text = "Gagal terhubung ke server. Kode: " .. code
      announceForAccessibility(pb_hasilText.text)
    end
  end)
end
pb_fetchButton.onClick = function()
  fetchPastebinData()
end
function resetPastebinDownloader()
  pb_urlInput.setText("")
  pb_hasilText.text = "Masukkan URL Pastebin dan tekan tombol ambil data."
end


-- =======================================
-- =======================================
-- LOGIKA 3: CARI RUTE (BARU)
-- =======================================
-- =======================================
function fetchRuteData()
  local fromText = rute_fromInput.text
  local toText = rute_toInput.text
  
  if fromText == "" or toText == "" then
    rute_hasilText.text = "Lokasi Asal dan Tujuan tidak boleh kosong!"
    announceForAccessibility(rute_hasilText.text)
    return
  end
  
  rute_hasilText.text = "Mencari rute dari " .. fromText .. " ke " .. toText .. "..."
  announceForAccessibility(rute_hasilText.text)
  vibrator.vibrate(50)
  
  -- Mengganti spasi dengan %20 agar aman untuk URL
  local encodedFrom = fromText:gsub(" ", "%%20")
  local encodedTo = toText:gsub(" ", "%%20")
  
  local fullApiUrl = API_RUTE .. "from=" .. encodedFrom .. "&to=" .. encodedTo
  
  Http.get(fullApiUrl, function(code, content)
    if code == 200 then
      local success, data = pcall(cjson.decode, content)
      
      if success and data and data.status == true and data.result then
        local res = data.result
        local formattedResult = "Rute Ditemukan:\n\n"
        
        -- Asal dan Tujuan
        formattedResult = formattedResult .. "ASAL:\n"
        formattedResult = formattedResult .. (res.asal.nama or "-") .. "\n"
        formattedResult = formattedResult .. (res.asal.alamat or "-") .. "\n\n"
        
        formattedResult = formattedResult .. "TUJUAN:\n"
        formattedResult = formattedResult .. (res.tujuan.nama or "-") .. "\n"
        formattedResult = formattedResult .. (res.tujuan.alamat or "-") .. "\n\n"
        
        -- Estimasi BBM
        if res.estimasi_biaya_bbm then
          formattedResult = formattedResult .. "ESTIMASI BBM:\n"
          formattedResult = formattedResult .. (res.estimasi_biaya_bbm.total_biaya or "-") .. " ("
          formattedResult = formattedResult .. (res.estimasi_biaya_bbm.total_liter or "-") .. " Liter)\n\n"
        end
        
        -- Peta
        formattedResult = formattedResult .. "LINK PETA:\n" .. (res.peta_statis or "Tidak tersedia") .. "\n\n"
        
        -- Arahan
        formattedResult = formattedResult .. "--- ARAHAN BELOKAN ---\n"
        if res.arah_penunjuk_jalan and #res.arah_penunjuk_jalan > 0 then
          for i, langkah in ipairs(res.arah_penunjuk_jalan) do
            formattedResult = formattedResult .. langkah.langkah .. ". " .. langkah.instruksi .. " (" .. langkah.jarak .. ")\n"
          end
        else
          formattedResult = formattedResult .. "Arahan tidak tersedia.\n"
        end
        
        rute_hasilText.text = formattedResult
        announceForAccessibility("Data rute berhasil dimuat.")
        
      elseif success and data and data.status == false then
         rute_hasilText.text = "Gagal mengambil data: " .. (data.message or "Server mengembalikan status false. Cek nama lokasi.")
         announceForAccessibility(rute_hasilText.text)
      else
        rute_hasilText.text = "Gagal memproses data JSON yang diterima."
        announceForAccessibility(rute_hasilText.text)
      end
    else
      rute_hasilText.text = "Gagal terhubung ke server. Kode: " .. code
      announceForAccessibility(rute_hasilText.text)
    end
  end)
end
rute_fetchButton.onClick = function()
  fetchRuteData()
end
function resetRute()
  rute_fromInput.setText("")
  rute_toInput.setText("")
  rute_hasilText.text = "Masukkan lokasi Asal dan Tujuan, lalu tekan 'Cari Rute'."
end

function fetchApkData()
local query = apk_searchInput.text
apk_result_container.removeAllViews()
if query == "" then
apk_statusText.text = "Nama APK tidak boleh kosong! Silakan isi kotak pencarian."
announceForAccessibility(apk_statusText.text)
return -- Hentikan fungsi jika kosong
end
apk_statusText.text = "Mencari APK untuk: " .. query .. "..."
announceForAccessibility(apk_statusText.text)
vibrator.vibrate(50)
local encodedQuery = query:gsub(" ", "%%20")
local fullApiUrl = API_APKPURE .. "?search=" .. encodedQuery
Http.get(fullApiUrl, function(code, content)
if code == 200 then
local success, data = pcall(cjson.decode, content)
if success and data and data.status == true and data.data then
local apps = data.data
if #apps > 0 then
apk_statusText.text = "Hasil Ditemukan:"
announceForAccessibility(apk_statusText.text)
for i, app in ipairs(apps) do
if app.title and app.title ~= "" then
local infoText = TextView(context)
local info = string.format(
"%d. %s\n Oleh: %s\n Rating: %s",
i,
app.title,
(app.developer or "-"),
(app.rating and app.rating.score or "-")
)
infoText.text = info
infoText.setTextColor(0xFFDDDDDD)
infoText.setTextIsSelectable(true) -- Agar info bisa disalin
local linkButton = Button(context)
linkButton.text = "Buka Link"
local params = LinearLayout.LayoutParams(
LinearLayout.LayoutParams.MATCH_PARENT,
LinearLayout.LayoutParams.WRAP_CONTENT
)
params.setMargins(0, 5, 0, 25) -- Margin Bawah 25px
linkButton.setLayoutParams(params)
local urlToOpen = app.link
linkButton.onClick = function()
vibrator.vibrate(30)
if urlToOpen then
local intent = Intent(Intent.ACTION_VIEW)
intent.setData(Uri.parse(urlToOpen))
pcall(function() context.startActivity(intent) end)
end
end
apk_result_container.addView(infoText)
apk_result_container.addView(linkButton)
end
end
else
apk_statusText.text = "Hasil tidak ditemukan untuk '" .. query .. "'."
end
elseif success and data and data.status == false then
apk_statusText.text = "Gagal mengambil data: " .. (data.message or "Server mengembalikan status false.")
else
apk_statusText.text = "Gagal memproses data JSON yang diterima."
end
announceForAccessibility(apk_statusText.text)
else
apk_statusText.text = "Gagal terhubung ke server. Kode: " .. code
announceForAccessibility(apk_statusText.text)
end
end)
end
apk_fetchButton.onClick = function()
fetchApkData()
end
function resetApk()
apk_searchInput.setText("") -- Hapus teks pencarian
apk_statusText.text = "Masukkan nama APK lalu tekan 'Cari APK'."
apk_result_container.removeAllViews() -- Bersihkan hasil
end

function fetchAn1Data()
local query = an1_searchInput.text
an1_result_container.removeAllViews()
if query == "" then
an1_statusText.text = "Nama APK tidak boleh kosong! Silakan isi kotak pencarian."
announceForAccessibility(an1_statusText.text)
return
end
an1_statusText.text = "Mencari di AN1 untuk: " .. query .. "..."
announceForAccessibility(an1_statusText.text)
vibrator.vibrate(50)
local encodedQuery = query:gsub(" ", "%%20")
local fullApiUrl = API_AN1 .. "?search=" .. encodedQuery
Http.get(fullApiUrl, function(code, content)
if code == 200 then
local success, data = pcall(cjson.decode, content)
if success and data and data.status == true and data.data then
local apps = data.data
if #apps > 0 then
an1_statusText.text = "Hasil Ditemukan:"
announceForAccessibility(an1_statusText.text)
for i, app in ipairs(apps) do
if app.title and app.title ~= "" then
local infoText = TextView(context)
local rating = (app.rating and app.rating.value or "-")
local info = string.format(
"%d. %s\n Oleh: %s\n Rating: %s",
i,
app.title,
(app.developer or "-"),
rating
)
infoText.text = info
infoText.setTextColor(0xFFDDDDDD)
infoText.setTextIsSelectable(true)
local linkButton = Button(context)
linkButton.text = "Buka Link"
local params = LinearLayout.LayoutParams(
LinearLayout.LayoutParams.MATCH_PARENT,
LinearLayout.LayoutParams.WRAP_CONTENT
)
params.setMargins(0, 5, 0, 25)
linkButton.setLayoutParams(params)
local urlToOpen = app.link
linkButton.onClick = function()
vibrator.vibrate(30)
if urlToOpen then
local intent = Intent(Intent.ACTION_VIEW)
intent.setData(Uri.parse(urlToOpen))
pcall(function() context.startActivity(intent) end)
dlg.dismiss()
end
end
an1_result_container.addView(infoText)
an1_result_container.addView(linkButton)
end
end
else
an1_statusText.text = "Hasil tidak ditemukan untuk '" .. query .. "'."
end
elseif success and data and data.status == false then
an1_statusText.text = "Gagal mengambil data: " .. (data.message or "Server mengembalikan status false.")
else
an1_statusText.text = "Gagal memproses data JSON yang diterima."
end
announceForAccessibility(an1_statusText.text)
else
an1_statusText.text = "Gagal terhubung ke server. Kode: " .. code
announceForAccessibility(an1_statusText.text)
end
end)
end
an1_fetchButton.onClick = function()
fetchAn1Data()
end
function resetAn1()
an1_searchInput.setText("")
an1_statusText.text = "Masukkan nama APK lalu tekan 'Cari di AN1'."
an1_result_container.removeAllViews()
end

function fetchApkmodyData()
local query = apkmody_searchInput.text
apkmody_result_container.removeAllViews()
if query == "" then
apkmody_statusText.text = "Nama APK tidak boleh kosong! Silakan isi kotak pencarian."
announceForAccessibility(apkmody_statusText.text)
return -- Hentikan fungsi jika kosong
end
apkmody_statusText.text = "Mencari di APKMody untuk: " .. query .. "..."
announceForAccessibility(apkmody_statusText.text)
vibrator.vibrate(50)
local encodedQuery = query:gsub(" ", "%%20")
local fullApiUrl = API_APKMODY .. "?search=" .. encodedQuery -- Menggunakan API_APKMODY
Http.get(fullApiUrl, function(code, content)
if code == 200 then
local success, data = pcall(cjson.decode, content)
if success and data and data.status == true and data.data then
local apps = data.data
if #apps > 0 then
apkmody_statusText.text = "Hasil Ditemukan:"
announceForAccessibility(apkmody_statusText.text)
for i, app in ipairs(apps) do
if app.title and app.title ~= "" then
local infoText = TextView(context)
local rating = (app.rating and app.rating.stars or "0")
local info = string.format(
"%d. %s\n Genre: %s\n Rating: %s Bintang",
i,
app.title,
(app.genre or "-"),
rating
)
infoText.text = info
infoText.setTextColor(0xFFDDDDDD)
infoText.setTextIsSelectable(true)
local linkButton = Button(context)
linkButton.text = "Buka Link"
local params = LinearLayout.LayoutParams(
LinearLayout.LayoutParams.MATCH_PARENT,
LinearLayout.LayoutParams.WRAP_CONTENT
)
params.setMargins(0, 5, 0, 25)
linkButton.setLayoutParams(params)
local urlToOpen = app.link
linkButton.onClick = function()
vibrator.vibrate(30)
if urlToOpen then
local intent = Intent(Intent.ACTION_VIEW)
intent.setData(Uri.parse(urlToOpen))
pcall(function() context.startActivity(intent) end)
dlg.dismiss()
end
end
apkmody_result_container.addView(infoText)
apkmody_result_container.addView(linkButton)
end
end
else
apkmody_statusText.text = "Hasil tidak ditemukan untuk '" .. query .. "'."
end
elseif success and data and data.status == false then
apkmody_statusText.text = "Gagal mengambil data: " .. (data.message or "Server mengembalikan status false.")
else
apkmody_statusText.text = "Gagal memproses data JSON yang diterima."
end
announceForAccessibility(apkmody_statusText.text)
else
apkmody_statusText.text = "Gagal terhubung ke server. Kode: " .. code
announceForAccessibility(apkmody_statusText.text)
end
end)
end
apkmody_fetchButton.onClick = function()
fetchApkmodyData()
end
function resetApkmody()
apkmody_searchInput.setText("")
apkmody_statusText.text = "Masukkan nama APK lalu tekan 'Cari di APKMody'."
apkmody_result_container.removeAllViews()
end

function fetchBeritaData()
	local query = berita_searchInput.text
	berita_result_container.removeAllViews()
	if query == "" then
		berita_statusText.text = "Kueri pencarian tidak boleh kosong!"
		announceForAccessibility(berita_statusText.text)
		return
	end
	berita_statusText.text = "Mencari berita untuk: " .. query .. "..."
	announceForAccessibility(berita_statusText.text)
	vibrator.vibrate(50)
	local encodedQuery = query:gsub(" ", "%%20")
	local fullApiUrl = API_BERITA .. "?prompt=" .. encodedQuery
	Http.get(fullApiUrl, function(code, content)
		if code == 200 then
			local success, data = pcall(cjson.decode, content)
			if success and data and data.search_results then
				local results = data.search_results
				if #results > 0 then
					berita_statusText.text = "Hasil Ditemukan:"
					announceForAccessibility(berita_statusText.text)
					for i, item in ipairs(results) do
						if item.title and item.url then
							local infoText = TextView(context)
							local info = string.format(
								"%d. %s\n   %s",
								i,
								item.title,
								(item.snippet or "-")
								)
							infoText.text = info
							infoText.setTextColor(0xFFDDDDDD)
							infoText.setTextIsSelectable(true)
							local linkButton = Button(context)
							linkButton.text = "Buka Link"
							local params = LinearLayout.LayoutParams(
								LinearLayout.LayoutParams.MATCH_PARENT,
								LinearLayout.LayoutParams.WRAP_CONTENT
								)
							params.setMargins(0, 5, 0, 25)
							linkButton.setLayoutParams(params)
							local urlToOpen = item.url
							linkButton.onClick = function()
							vibrator.vibrate(30)
							if urlToOpen then
								local intent = Intent(Intent.ACTION_VIEW)
								intent.setData(Uri.parse(urlToOpen))
								pcall(function() context.startActivity(intent) end)
								dlg.dismiss()
							end
						end
						berita_result_container.addView(infoText)
						berita_result_container.addView(linkButton)
					end
				end
			else
				berita_statusText.text = "Hasil tidak ditemukan untuk '" .. query .. "'."
			end
		elseif success and data and data.status == false then
			berita_statusText.text = "Gagal mengambil data: " .. (data.message or "Server mengembalikan status false.")
		else
			berita_statusText.text = "Gagal memproses data JSON yang diterima."
		end
		announceForAccessibility(berita_statusText.text)
	else
		berita_statusText.text = "Gagal terhubung ke server. Kode: " .. code
		announceForAccessibility(berita_statusText.text)
	end
	end)
end
berita_fetchButton.onClick = function()
fetchBeritaData()
end
function resetBerita()
	berita_searchInput.setText("")
	berita_statusText.text = "Masukkan kueri lalu tekan 'Cari Berita'."
	berita_result_container.removeAllViews()
end

local tools = {"Gist Downloader", "Pastebin Downloader", "Cari Rute", "Cari apk di apk pure", "Cari apk di AN1", "Cari  APKMody", "Cari Berita"}

local adapter = ArrayAdapter(context, R.layout.simple_spinner_item, tools)
adapter.setDropDownViewResource(R.layout.simple_spinner_dropdown_item)
toolSpinner.adapter = adapter

toolSpinner.onItemSelectedListener = luajava.createProxy("android.widget.AdapterView$OnItemSelectedListener", {
		onItemSelected = function(parent, view, position, id)

		gist_layout.visibility = View.GONE
		pb_layout.visibility = View.GONE
		rute_layout.visibility = View.GONE
		apk_layout.visibility = View.GONE
		an1_layout.visibility = View.GONE
		apkmody_layout.visibility = View.GONE
    berita_layout.visibility = View.GONE

		if position == 0 then
			gist_layout.visibility = View.VISIBLE
			resetGistDownloader()

		elseif position == 1 then
			pb_layout.visibility = View.VISIBLE
			resetPastebinDownloader()

    elseif position == 6 then -- Cari Berita
      berita_layout.visibility = View.VISIBLE
      resetBerita()

		elseif position == 2 then
			rute_layout.visibility = View.VISIBLE
			resetRute()

		elseif position == 3 then
			apk_layout.visibility = View.VISIBLE
			resetApk()

		elseif position == 4 then
			an1_layout.visibility = View.VISIBLE
			resetAn1()

		elseif position == 5 then
			apkmody_layout.visibility = View.VISIBLE
			resetApkmody()

		end
		end,

		onNothingSelected = function(parent)

		end
		})

exitButton.onClick = function()
pcall(function() mainHandler:removeCallbacksAndMessages(nil) end)
dlg.dismiss()
end

dlg.show()
