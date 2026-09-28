import asyncio
import re
import sys
import os
from datetime import datetime, timezone
import syncedlyrics
from winrt.windows.media.control import GlobalSystemMediaTransportControlsSessionManager as MediaManager

# Kode warna ANSI untuk Biru dan Reset
BIRU = '\033[94m'
RESET = '\033[0m'

# Fungsi untuk animasi mengetik huruf per huruf
async def ketik_animasi(teks, kecepatan):
    for huruf in teks:
        sys.stdout.write(f"{BIRU}{huruf}{RESET}")
        sys.stdout.flush()
        await asyncio.sleep(kecepatan)
    sys.stdout.write('\n')
    sys.stdout.flush()

# Fungsi mengambil lagu & waktu Real-Time
async def get_current_media_info():
    sessions = await MediaManager.request_async()
    current_session = sessions.get_current_session()
    
    if current_session:
        info = await current_session.try_get_media_properties_async()
        title = info.title
        artist = info.artist
        
        timeline = current_session.get_timeline_properties()
        playback = current_session.get_playback_info()
        
        position_seconds = timeline.position.total_seconds()
        
        if playback and playback.playback_status == 4: # 4 = 'Playing'
            last_updated = timeline.last_updated_time
            if last_updated:
                sekarang = datetime.now(timezone.utc)
                selisih = (sekarang - last_updated).total_seconds()
                position_seconds += selisih
        
        return f"{title} {artist}", position_seconds
    return None, 0

# Fungsi memecah format [00:15.20] menjadi detik
def parse_lrc(lrc_text):
    if not lrc_text: return []
    lines = lrc_text.split('\n')
    parsed_lyrics = []
    
    for line in lines:
        match = re.match(r'\[(\d+):(\d+\.\d+)\](.*)', line)
        if match:
            mins = int(match.group(1))
            secs = float(match.group(2))
            text = match.group(3).strip()
            
            if text: 
                total_secs = (mins * 60) + secs
                parsed_lyrics.append({"time": total_secs, "text": text})
                
    return parsed_lyrics

async def main():
    os.system('') # Aktifkan mode warna ANSI di Windows

    lagu_sebelumnya = ""
    lirik_aktif = []
    index_lirik_sekarang = -1

    print("Lirik")
    print("-" * 40)

    while True:
        lagu_sekarang, detik_berjalan = await get_current_media_info()

        if lagu_sekarang:
            if lagu_sekarang != lagu_sebelumnya:
                print(f"\n Lirik Lagu: {lagu_sekarang}")
                
                lrc_mentah = syncedlyrics.search(lagu_sekarang)
                lirik_aktif = parse_lrc(lrc_mentah)
                
                lagu_sebelumnya = lagu_sekarang
                index_lirik_sekarang = -1 
                
                if not lirik_aktif:
                    print("(Lirik tidak ditemukan di database)")

            if lirik_aktif:
                for i, baris in enumerate(lirik_aktif):
                    if detik_berjalan >= baris["time"] and i > index_lirik_sekarang:
                        
                        teks_tampil = f">> {baris['text']}"
                        
                        # --- FITUR KECEPATAN DINAMIS ---
                        # 1. Hitung sisa waktu sebelum baris lirik berikutnya muncul
                        if i + 1 < len(lirik_aktif):
                            waktu_tersedia = lirik_aktif[i+1]["time"] - baris["time"]
                        else:
                            waktu_tersedia = 4.0 # Baris terakhir diberi waktu default 4 detik
                            
                        # 2. Rumus: (80% waktu tersedia) dibagi (jumlah huruf). 80% agar tidak terlalu mepet.
                        if len(teks_tampil) > 0:
                            kecepatan_hitung = (waktu_tersedia * 0.8) / len(teks_tampil)
                        else:
                            kecepatan_hitung = 0.25
                            
                        # 3. Batasi kecepatan: minimal 0.01 (sangat ngebut), maksimal 0.15 (santai)
                        kecepatan_dinamis = max(0.01, min(0.15, kecepatan_hitung))
                        # -------------------------------
                        
                        await ketik_animasi(teks_tampil, kecepatan=kecepatan_dinamis)
                        index_lirik_sekarang = i
                        
        else:
            if lagu_sebelumnya != "":
                print("\n[Musik Berhenti]")
                lagu_sebelumnya = ""

        await asyncio.sleep(0.1)

if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\n\nByee!")
