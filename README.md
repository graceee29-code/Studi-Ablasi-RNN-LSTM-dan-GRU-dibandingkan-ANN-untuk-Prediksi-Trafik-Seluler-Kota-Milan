# Studi-Ablasi-RNN-LSTM-dan-GRU-dibandingkan-ANN-untuk-Prediksi-Trafik-Seluler-Kota-Milan
\documentclass[conference,a4paper]{IEEEtran}
\IEEEoverridecommandlockouts

\usepackage[indonesian]{babel}
\usepackage[utf8]{inputenc}
\usepackage{mathptmx}
\usepackage{cite}
\usepackage{amsmath,amssymb}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{siunitx}
\usepackage{xcolor}
\usepackage[hidelinks]{hyperref}
\graphicspath{{gambar/}}

% Penanda petunjuk (hapus pemakaiannya di versi akhir)
\newcommand{\petunjuk}[1]{\textcolor{gray}{[#1]}}
\newcommand{\isi}[1]{\textcolor{red}{[#1]}}

\begin{document}

\title{Studi Ablasi RNN, LSTM, dan GRU dibandingkan ANN untuk Prediksi Trafik Seluler Kota Milan}

% Satu blok per anggota (maks. 3). Hapus blok yang tidak dipakai.
\author{
\IEEEauthorblockN{Grace Victoria Simamora}
\IEEEauthorblockA{NIM 124400081\\
Teknik Telekomunikasi, ITERA\\
grace.124400081@student.itera.ac.id}
\and
\IEEEauthorblockN{Justianaa Insyirah}
\IEEEauthorblockA{NIM 124400085\\
Teknik Telekomunikasi, ITERA\\
justianaa.124400085@student.itera.ac.id}
\and
\IEEEauthorblockN{Tantri Nur Sayfi}
\IEEEauthorblockA{NIM 124400091\\
Teknik Telekomunikasi, ITERA\\
tantri.124400091@student.itera.ac.id}
}

\maketitle

\begin{abstract}
Prediksi trafik internet per jam penting untuk memahami pola penggunaan jaringan telekomunikasi dan mendukung pengelolaan kapasitas jaringan. Penelitian ini membandingkan \textit{Artificial Neural Network} (ANN), \textit{Recurrent Neural Network} (RNN), \textit{Long Short-Term Memory} (LSTM), dan \textit{Gated Recurrent Unit} (GRU) menggunakan dataset Telecom Italia. Evaluasi dilakukan pada kotak Duomo di Kota Milan dan pengujian generalisasi pada lokasi kelompok, yaitu Navigli (Darsena). Studi ablasi menguji lima faktor utama: panjang jendela input, jumlah neuron tersembunyi, jumlah lapisan, \textit{learning rate}, dan kelompok fitur. Dua ablasi tambahan menguji \textit{dropout} dan \textit{gradient clipping} pada \textit{learning rate} 0,01. Setiap konfigurasi dievaluasi menggunakan tiga \textit{seed} dengan metrik \textit{Mean Absolute Error} (MAE). Pada pengujian acuan di Duomo, RNN memperoleh test MAE sebesar $229{,}6\pm5{,}1$, lebih rendah $27{,}0\%$ dibandingkan ANN dengan MAE $314{,}3\pm 10{,}4$. Pada lokasi Navigli (Darsena), GRU memperoleh MAE terendah sebesar $190{,}8\pm6{,}8$, menunjukkan bahwa kinerja model dapat berbeda antarwilayah. Hasil ini menegaskan bahwa pemilihan arsitektur dan konfigurasi perlu mempertimbangkan karakteristik lokasi serta kestabilan hasil pengujian.
\end{abstract}

\begin{IEEEkeywords}
 prediksi trafik internet, Telecom Italia, ANN, RNN, LSTM, GRU, ablasi.
\end{IEEEkeywords}

% ---------------------------------------------------------------------
\section{Pendahuluan}
Prediksi trafik seluler diperlukan untuk membantu operator memahami pola penggunaan jaringan, memperkirakan kebutuhan kapasitas, dan mengoptimalkan alokasi sumber daya. Metode Artificial Neural Network (ANN) dapat mempelajari hubungan antara masukan dan keluaran, tetapi tidak secara langsung mempertahankan urutan informasi temporal. Sebaliknya, Recurrent Neural Network (RNN) memanfaatkan informasi dari langkah waktu sebelumnya, sedangkan Long Short-Term Memory (LSTM) dan Gated Recurrent Unit (GRU) menggunakan mekanisme gerbang untuk mengatur informasi yang dipertahankan selama pemrosesan sekuens \cite{hochreiter1997lstm,cho2014gru}.

Studi ablasi dilakukan dengan mengubah satu faktor pada satu waktu untuk mengetahui pengaruh konfigurasi model terhadap akurasi prediksi. Kontribusi penelitian ini adalah membandingkan kinerja ANN, RNN, LSTM, dan GRU, menganalisis pengaruh lima hyperparameter utama dan dua faktor tambahan, serta mengevaluasi kemampuan generalisasi model pada lokasi berbeda di Kota Milan. Mengapa model sekuensial (RNN, LSTM \cite{hochreiter1997lstm}, GRU \cite{cho2014gru}) dibandingkan dengan ANN? Apa itu studi ablasi dan mengapa penting? Tutup dengan daftar kontribusi laporan ini (2--3 butir). Maks. 0,5 kolom.

% ---------------------------------------------------------------------
\section{Data dan Metode}

\subsection{Dataset}
Penelitian ini menggunakan dataset Telecom Italia Big Data Challenge yang memuat aktivitas komunikasi seluler di Kota Milan \cite{barlacchi2015milan}. Data aktivitas jaringan diolah menjadi deret waktu per jam untuk mempelajari pola trafik dan membangun model prediksi. Lokasi acuan penelitian adalah kotak Duomo dengan kode 5060, sedangkan pengujian generalisasi dilakukan pada lokasi Navigli (Darsena). Pemilihan dua lokasi ini bertujuan untuk mengetahui apakah model yang memiliki kinerja baik pada lokasi acuan juga mampu memprediksi trafik pada wilayah dengan karakteristik aktivitas yang berbeda. Telecom Italia Big Data Challenge \cite{barlacchi2015milan}, versi per jam dari \cite{zhang2021fedda}: periode, resolusi, fitur. Sebutkan kotak Duomo (5060) dan lokasi kelompok (kode, nama, nomor kotak). Gambar~\ref{fig:peta} boleh dipakai.

\begin{figure}[t]
  \centering
  \includegraphics[width=0.85\linewidth]{peta_lokasi_tugas.png}
  \caption{Rata-rata aktivitas internet per kotak di Milan. Lingkaran kuning: lokasi tugas (kode 0--9); lingkaran toska: Duomo.}
  \label{fig:peta}
\end{figure}

\subsection{Pembagian Data dan Praproses}
Data dibagi secara kronologis menjadi data pelatihan, validasi, pengujian, dan periode Natal. Pembagian berdasarkan waktu diperlukan agar model tidak menggunakan informasi dari masa depan ketika mempelajari pola trafik masa lalu. Standarisasi dilakukan menggunakan statistik data pelatihan sebelum periode validasi untuk menghindari kebocoran informasi. Selanjutnya, data disusun menggunakan metode windowing, yaitu mengambil sejumlah sampel waktu sebelumnya sebagai masukan untuk memprediksi trafik satu jam berikutnya. Pembagian berdasarkan waktu (train / val / test / Natal), standarisasi dengan statistik sebelum minggu validasi, \textit{windowing}. Jelaskan mengapa data deret waktu tidak boleh diacak.

\subsection{Model dan Konfigurasi Acuan}
Penelitian ini membandingkan empat arsitektur jaringan saraf. ANN mempelajari hubungan antara fitur masukan dan keluaran melalui lapisan neuron. RNN memproses data secara berurutan dengan memanfaatkan informasi dari langkah waktu sebelumnya. LSTM menggunakan gerbang untuk mengatur penyimpanan dan penghapusan informasi, sedangkan GRU menerapkan mekanisme gerbang dengan struktur yang lebih sederhana. Seluruh model menggunakan optimizer Adam dengan learning rate $10^{-3}$ dan batch size 64. Panjang jendela masukan adalah 24 jam dengan horizon prediksi satu jam. Ukuran hidden dan jumlah lapisan disesuaikan dengan konfigurasi acuan pada Tabel~\ref{tab:acuan}.

\begin{table}[t]
  \centering
  \caption{Konfigurasi acuan}
  \label{tab:acuan}
  \begin{tabular}{@{}ll@{}}
    \toprule
    Hyperparameter & Nilai acuan \\
    \midrule
    Panjang jendela $W$ & 24 jam \\
    Horizon & 1 jam \\
    Ukuran hidden & ANN 32, RNN 68, LSTM 33, GRU 38 \\
    Jumlah lapisan & ANN 2, rekuren 1 \\
    Fitur & net, call, sms, jam, hari (7) \\
    Optimizer, \textit{learning rate} & Adam, $10^{-3}$ \\
    \textit{Batch size} & 64 \\
    \textit{Early stopping} & \textit{patience} 30, maks.\ 300 epoch \\
    \bottomrule
  \end{tabular}
\end{table}

\subsection{Protokol Ablasi dan Evaluasi}
Eksperimen ablasi dilakukan dengan mengubah satu faktor pada setiap percobaan, meliputi panjang jendela, ukuran hidden, jumlah lapisan, learning rate, dan kelompok fitur. Dua eksperimen tambahan dilakukan dengan mengubah dropout dan gradient clipping. Setiap konfigurasi dijalankan menggunakan tiga seed untuk mengevaluasi kestabilan hasil. Kinerja model diukur menggunakan Mean Absolute Error (MAE), Root Mean Squared Error (RMSE), dan Mean Absolute Percentage Error (MAPE) pada skala asli. Pemilihan konfigurasi terbaik didasarkan pada nilai MAE validasi, sedangkan hasil pengujian digunakan untuk menilai kemampuan prediksi model pada data yang belum pernah dilihat selama pelatihan. Metrik: MAE, RMSE, MAPE pada skala asli. Pemilihan konfigurasi terbaik memakai val MAE. Sebutkan perangkat yang dipakai (CPU/GPU) dan total waktu training.
% =====================================================================
% BAGIAN HASIL, PEMBAHASAN, KESIMPULAN
% Tempel menggantikan semua isi dari \section{Hasil} sampai sebelum
% \section*{Kontribusi Anggota} di main.tex.
%
% WAJIB: tambahkan satu baris ini di preamble main.tex (dekat \petunjuk):
%   \newcommand{\isi}[1]{\textcolor{red}{[#1]}}
% Teks merah [ ... ] = bagian yang harus kalian isi/cek sendiri, lalu hapus.
% Semua angka berasal dari hasil/ablasi.csv, baseline_*.csv, C_*.csv, D_*.csv
% (rata-rata dan simpangan baku sampel dari 3 seed).
% =====================================================================
 
\section{Sebelum eksperimen dilakukan, dropout diperkirakan dapat mengurangi overfitting dengan membatasi ketergantungan model pada neuron tertentu. Gradient clipping diperkirakan dapat membantu menjaga kestabilan pembaruan bobot dengan membatasi besar gradien selama pelatihan.}
 
\subsection{Baseline Naif dan Acuan}
Tabel~\ref{tab:acuan-duomo} membandingkan konfigurasi acuan dengan tiga baseline naif pada horizon 1 jam di kotak Duomo. Pada minggu test, RNN memperoleh MAE terendah ($229{,}6 \pm 5{,}1$), disusul GRU ($237{,}2 \pm 9{,}3$), LSTM ($273{,}6 \pm 25{,}4$), dan ANN ($314{,}3 \pm 10{,}4$). Dibandingkan baseline naif terbaik (nilai minggu lalu, MAE $467{,}6$), RNN menurunkan galat sekitar $50{,}9\%$ dan ANN sekitar $32{,}8\%$. Pada minggu Natal, seluruh model jauh lebih buruk daripada pada minggu test, dan baseline ``nilai terakhir'' (MAE $431{,}2$) hanya dikalahkan tipis oleh RNN ($421{,}2 \pm 2{,}4$).
 
\begin{table}[t]
  \centering
  \caption{Acuan dan baseline naif di Duomo (horizon 1 jam). Model: rata-rata $\pm$ simpangan baku, 3 \textit{seed}; baseline naif deterministik.}
  \label{tab:acuan-duomo}
  \scriptsize
  \setlength{\tabcolsep}{2.5pt}
  \begin{tabular}{@{}lcccc@{}}
    \toprule
    Model & Val MAE & Test MAE & Test MAPE (\%) & Natal MAE \\
    \midrule
    Naif: nilai terakhir & 662,6 & 600,9 & 18,4 & 431,2 \\
    Naif: kemarin        & 595,3 & 552,2 & 23,8 & 921,5 \\
    Naif: minggu lalu    & 492,8 & 467,6 & 19,9 & 1426,2 \\
    \midrule
    ANN  & $249{,}7 \pm 8{,}6$  & $314{,}3 \pm 10{,}4$ & $11{,}8 \pm 1{,}1$ & $674{,}1 \pm 41{,}8$ \\
    RNN  & $206{,}1 \pm 7{,}4$  & $\mathbf{229{,}6 \pm 5{,}1}$ & $8{,}5 \pm 0{,}3$ & $\mathbf{421{,}2 \pm 2{,}4}$ \\
    LSTM & $230{,}5 \pm 12{,}2$ & $273{,}6 \pm 25{,}4$ & $9{,}1 \pm 1{,}0$ & $586{,}0 \pm 56{,}8$ \\
    GRU  & $207{,}5 \pm 3{,}8$  & $237{,}2 \pm 9{,}3$ & $8{,}0 \pm 0{,}7$ & $468{,}2 \pm 21{,}8$ \\
    \bottomrule
  \end{tabular}
\end{table}
 
\subsection{Ablasi Wajib}
Tabel~\ref{tab:a13} dan Tabel~\ref{tab:a45} merangkum lima faktor wajib; Gambar~\ref{fig:a} menampilkan dua di antaranya.
 
\textbf{A1 (panjang jendela).} ANN memburuk seiring jendela memanjang, dari $268{,}7 \pm 18{,}1$ ($W=6$) menjadi $402{,}0 \pm 12{,}5$ ($W=168$). Model rekuren hampir tidak berubah: RNN berkisar $224{,}5$--$233{,}5$, dengan selisih yang sebanding dengan simpangan bakunya.
 
\textbf{A2 (ukuran hidden).} Hidden yang lebih besar cenderung membantu model berpintu: LSTM turun dari $290{,}8 \pm 20{,}9$ (hidden 32) ke $245{,}6 \pm 6{,}7$ (hidden 128) dan GRU dari $232{,}7 \pm 5{,}1$ ke $220{,}3 \pm 10{,}6$. Perbaikan GRU masih berada dalam kisaran satu simpangan baku.
 
\textbf{A3 (jumlah lapisan).} Menambah lapisan tidak membantu. LSTM memburuk tajam pada tiga lapisan ($412{,}6 \pm 54{,}2$ dibandingkan $273{,}6 \pm 25{,}4$ pada satu lapisan), dan GRU juga memburuk ($264{,}2 \pm 11{,}4$ dibandingkan $237{,}2 \pm 9{,}3$). Perbedaan RNN antara satu dan dua lapisan ($229{,}6$ dan $228{,}4$) dapat diabaikan.
 
\textbf{A4 (\textit{learning rate}).} Faktor ini paling berpengaruh. Pada $10^{-4}$ semua model buruk dan seluruh run berhenti di batas 300 epoch (belum konvergen); GRU mencapai $356{,}3 \pm 7{,}9$. Pada $10^{-2}$, LSTM membaik menjadi $225{,}8 \pm 4{,}2$ dan GRU menjadi $215{,}7 \pm 10{,}9$, sedangkan ANN menjadi tidak stabil (simpangan baku $53{,}2$).
 
\textbf{A5 (kelompok fitur).}
Pola berbeda antara ANN dan model rekuren.
ANN justru terbaik dengan fitur \texttt{net} saja
($272{,}0 \pm 13{,}8$), sedangkan RNN terbaik
dengan \texttt{net+call+sms} ($227{,}6 \pm 2{,}2$)
dan jauh lebih baik daripada \texttt{net} saja
($254{,}1 \pm 1{,}7$). Fitur waktu tidak memberi
perbaikan yang konsisten.
 
\begin{figure*}[t]
  \centering
  \begin{minipage}{0.48\textwidth}
    \centering
    \includegraphics[width=\linewidth]{A1_W.png}
  \end{minipage}\hfill
  \begin{minipage}{0.48\textwidth}
    \centering
    \includegraphics[width=\linewidth]{A4_lr.png}
  \end{minipage}
  \caption{Ablasi A1 (panjang jendela, kiri) dan A4 (\textit{learning rate}, kanan) di Duomo.}
  \label{fig:a}
\end{figure*}
 
\subsection{Ablasi Pilihan}
{Kami memilih B2 (\textit{dropout}) dan B7 (\textit{gradient clipping}).
\textbf{Hipotesis B2 (Dropout):}
Kami memperkirakan bahwa penambahan dropout dapat mengurangi
overfitting pada model ANN dan RNN, sehingga model mampu
melakukan generalisasi lebih baik pada data pengujian.
Namun, dropout yang terlalu besar dapat menurunkan akurasi model.

\textbf{Hipotesis B7 (Gradient Clipping):}
Kami memperkirakan bahwa penerapan gradient clipping pada
model RNN dapat membantu mengatasi masalah exploding gradients,
membuat proses pelatihan lebih stabil, dan berpotensi meningkatkan
kinerja prediksi. Namun, pengaruhnya terhadap hasil akhir
bergantung pada nilai batas gradien yang digunakan.}
 
\textbf{B2.} Dropout 0,2 hanya membantu GRU ($225{,}0 \pm 12{,}8$ dari $237{,}2 \pm 9{,}3$) dan LSTM ($264{,}6 \pm 16{,}6$ dari $273{,}6 \pm 25{,}4$), dan keduanya masih dalam kisaran simpangan baku. Dropout 0,2 memperburuk RNN ($260{,}3 \pm 13{,}1$) dan membuat ANN tidak stabil ($330{,}3 \pm 46{,}9$). Dropout 0,5 merugikan semua model, terutama ANN ($411{,}6 \pm 8{,}0$).
 
\textbf{B7.} Pada $\textit{learning rate}=10^{-2}$, \textit{clipping} 0,5 dan 1,0 tidak mengubah galat secara berarti: LSTM $231{,}6 \pm 3{,}8$ dan $226{,}4 \pm 2{,}1$ terhadap $225{,}8 \pm 4{,}2$ tanpa \textit{clipping}, GRU $218{,}9$ dan $223{,}0$ terhadap $215{,}7$. Satu efek yang tampak adalah ANN yang kehilangan sebagian ketidakstabilan (simpangan baku turun dari $53{,}2$ menjadi $26{,}5$ dan $13{,}6$).
 
\begin{table*}[t]
  \centering
  \caption{Ablasi wajib A1--A3 di Duomo: test MAE (rata-rata $\pm$ simpangan baku sampel, 3 \textit{seed}). Cetak tebal: terendah per baris.}
  \label{tab:a13}
  \footnotesize
  \begin{tabular}{@{}lcccc@{}}
    \toprule
    Nilai faktor & ANN & RNN & LSTM & GRU \\
    \midrule
    \multicolumn{5}{@{}l}{\textit{A1: panjang jendela W (jam)}} \\
    \midrule
    6 & $268{,}7 \pm 18{,}1$ & ${\mathbf{228{,}6 \pm 5{,}4}}$ & $276{,}3 \pm 14{,}7$ & $249{,}8 \pm 1{,}4$ \\
    12 & $307{,}8 \pm 13{,}1$ & ${\mathbf{233{,}2 \pm 11{,}2}}$ & $302{,}7 \pm 17{,}4$ & $246{,}1 \pm 13{,}6$ \\
    24 (acuan) & $314{,}3 \pm 10{,}4$ & ${\mathbf{229{,}6 \pm 5{,}1}}$ & $273{,}6 \pm 25{,}4$ & $237{,}2 \pm 9{,}3$ \\
    48 & $333{,}1 \pm 20{,}3$ & ${\mathbf{224{,}5 \pm 11{,}7}}$ & $304{,}4 \pm 27{,}1$ & $237{,}1 \pm 10{,}6$ \\
    168 & $402{,}0 \pm 12{,}5$ & ${\mathbf{233{,}5 \pm 4{,}8}}$ & $292{,}8 \pm 20{,}0$ & $237{,}2 \pm 10{,}6$ \\
    \midrule
    \multicolumn{5}{@{}l}{\textit{A2: ukuran hidden}} \\
    \midrule
    16 & $313{,}2 \pm 19{,}0$ & $253{,}3 \pm 9{,}3$ & $295{,}3 \pm 13{,}1$ & ${\mathbf{246{,}9 \pm 10{,}4}}$ \\
    32 & $314{,}3 \pm 10{,}4$ & $245{,}2 \pm 3{,}7$ & $290{,}8 \pm 20{,}9$ & ${\mathbf{232{,}7 \pm 5{,}1}}$ \\
    64 & $316{,}2 \pm 24{,}2$ & $240{,}9 \pm 15{,}0$ & $269{,}2 \pm 12{,}7$ & ${\mathbf{225{,}0 \pm 12{,}7}}$ \\
    128 & $296{,}4 \pm 32{,}0$ & $234{,}6 \pm 18{,}6$ & $245{,}6 \pm 6{,}7$ & ${\mathbf{220{,}3 \pm 10{,}6}}$ \\
    \midrule
    \multicolumn{5}{@{}l}{\textit{A3: jumlah lapisan}} \\
    \midrule
    1 & $311{,}7 \pm 10{,}8$ & ${\mathbf{229{,}6 \pm 5{,}1}}$ & $273{,}6 \pm 25{,}4$ & $237{,}2 \pm 9{,}3$ \\
    2 & $314{,}3 \pm 10{,}4$ & ${\mathbf{228{,}4 \pm 8{,}2}}$ & $308{,}9 \pm 20{,}6$ & $239{,}2 \pm 6{,}4$ \\
    3 & $299{,}8 \pm 25{,}1$ & ${\mathbf{245{,}5 \pm 19{,}1}}$ & $412{,}6 \pm 54{,}2$ & $264{,}2 \pm 11{,}4$ \\
    \bottomrule
  \end{tabular}
\end{table*}
 
\begin{table*}[t]
  \centering
  \caption{Ablasi wajib A4--A5 di Duomo: test MAE (rata-rata $\pm$ simpangan baku sampel, 3 \textit{seed}).}
  \label{tab:a45}
  \footnotesize
  \begin{tabular}{@{}lcccc@{}}
    \toprule
    Nilai faktor & ANN & RNN & LSTM & GRU \\
    \midrule
    \multicolumn{5}{@{}l}{\textit{A4: learning rate}} \\
    \midrule
    $10^{-4}$ & $339{,}7 \pm 22{,}5$ & ${\mathbf{287{,}9 \pm 7{,}3}}$ & $373{,}5 \pm 14{,}3$ & $356{,}3 \pm 7{,}9$ \\
    $10^{-3}$ (acuan) & $314{,}3 \pm 10{,}4$ & ${\mathbf{229{,}6 \pm 5{,}1}}$ & $273{,}6 \pm 25{,}4$ & $237{,}2 \pm 9{,}3$ \\
    $10^{-2}$ & $309{,}4 \pm 53{,}2$ & $239{,}4 \pm 6{,}1$ & $225{,}8 \pm 4{,}2$ & ${\mathbf{215{,}7 \pm 10{,}9}}$ \\
    \midrule
    \multicolumn{5}{@{}l}{\textit{A5: kelompok fitur}} \\
    \midrule
    net & $272{,}0 \pm 13{,}8$ & ${\mathbf{254{,}1 \pm 1{,}7}}$ & $284{,}1 \pm 19{,}3$ & $270{,}5 \pm 10{,}9$ \\
    net+call+sms & $317{,}5 \pm 9{,}7$ & ${\mathbf{227{,}6 \pm 2{,}2}}$ & $262{,}6 \pm 10{,}9$ & $250{,}7 \pm 9{,}3$ \\
    net+waktu & $296{,}4 \pm 9{,}1$ & $244{,}9 \pm 22{,}1$ & $292{,}6 \pm 25{,}1$ & ${\mathbf{239{,}7 \pm 25{,}4}}$ \\
    semua (acuan) & $314{,}3 \pm 10{,}4$ & ${\mathbf{229{,}6 \pm 5{,}1}}$ & $273{,}6 \pm 25{,}4$ & $237{,}2 \pm 9{,}3$ \\
    \bottomrule
  \end{tabular}
\end{table*}
 
\begin{table}[t]
  \centering
  \caption{Ablasi pilihan di Duomo: test MAE (rata-rata $\pm$ simpangan baku, 3 \textit{seed}). B7 dijalankan pada $\textit{learning rate}=10^{-2}$.}
  \label{tab:b}
  \scriptsize
  \setlength{\tabcolsep}{2.5pt}
  \begin{tabular}{@{}lcccc@{}}
    \toprule
    Nilai & ANN & RNN & LSTM & GRU \\
    \midrule
    \multicolumn{5}{@{}l}{\textit{B2: dropout}} \\
    \midrule
    0 (acuan) & $314{,}3 \pm 10{,}4$ & ${\mathbf{229{,}6 \pm 5{,}1}}$ & $273{,}6 \pm 25{,}4$ & $237{,}2 \pm 9{,}3$ \\
    0,2 & $330{,}3 \pm 46{,}9$ & $260{,}3 \pm 13{,}1$ & $264{,}6 \pm 16{,}6$ & ${\mathbf{225{,}0 \pm 12{,}8}}$ \\
    0,5 & $411{,}6 \pm 8{,}0$ & ${\mathbf{260{,}2 \pm 6{,}4}}$ & $289{,}1 \pm 10{,}3$ & $273{,}6 \pm 20{,}5$ \\
    \midrule
    \multicolumn{5}{@{}l}{\textit{B7: gradient clipping}} \\
    \midrule
    tanpa & $309{,}4 \pm 53{,}2$ & $239{,}4 \pm 6{,}1$ & $225{,}8 \pm 4{,}2$ & ${\mathbf{215{,}7 \pm 10{,}9}}$ \\
    0,5 & $314{,}1 \pm 26{,}5$ & $235{,}6 \pm 15{,}7$ & $231{,}6 \pm 3{,}8$ & ${\mathbf{218{,}9 \pm 18{,}8}}$ \\
    1,0 & $334{,}9 \pm 13{,}6$ & $231{,}3 \pm 9{,}0$ & $226{,}4 \pm 2{,}1$ & ${\mathbf{223{,}0 \pm 6{,}2}}$ \\
    \bottomrule
  \end{tabular}
\end{table}
 
\subsection{Konfigurasi Terbaik}
\isi Berdasarkan eksperimen Bagian C, konfigurasi terbaik setiap model ditentukan berdasarkan nilai rata-rata validation MAE dari eksperimen Bagian A dan B. Konfigurasi yang terpilih kemudian dibandingkan dengan konfigurasi acuan menggunakan test MAE dan Natal MAE.
Hasil perbandingan pada dataset Duomo menunjukkan bahwa perubahan
konfigurasi tidak selalu meningkatkan kinerja seluruh model. Beberapa
model dapat mengalami penurunan test MAE, sedangkan model lainnya
dapat mengalami peningkatan kesalahan prediksi. Oleh karena itu,
pemilihan konfigurasi terbaik berdasarkan validation MAE perlu
dievaluasi kembali menggunakan data pengujian.
Pada dataset Navigli, kinerja konfigurasi terbaik juga perlu
dibandingkan dengan konfigurasi acuan untuk mengetahui apakah
peningkatan kinerja dapat dipertahankan pada lokasi yang berbeda.
Perbedaan pola trafik dan interaksi antarhyperparameter dapat
memengaruhi hasil prediksi, terutama pada periode Natal.
Dengan demikian, konfigurasi terbaik ditetapkan berdasarkan hasil
eksperimen yang telah diverifikasi, bukan hanya berdasarkan dugaan
atau hasil dari satu faktor ablasi.
 
\subsection{Uji Generalisasi}
Tabel~\ref{tab:acuan-navigli} menunjukkan hasil acuan di Navigli (Darsena). GRU memperoleh test MAE terendah ($190{,}8 \pm 6{,}8$), tetapi RNN ($193{,}0 \pm 9{,}7$) praktis setara dan justru memiliki val MAE terendah ($147{,}3 \pm 4{,}9$). Urutan model berubah dibandingkan Duomo: ANN ($222{,}3 \pm 27{,}4$) kini mengungguli LSTM ($247{,}3 \pm 54{,}3$), dan selisih ANN terhadap model terbaik mengecil. Semua model mengalahkan baseline naif terbaik pada minggu test (MAE $272{,}0$). Sebaliknya, pada minggu Natal seluruh model ($496{,}4$ sampai $950{,}8$) jauh lebih buruk daripada baseline ``nilai terakhir'' ($177{,}7$). \isi 
Setelah perbaikan eksperimen Bagian C, konfigurasi terbaik pada
dataset Navigli ditentukan berdasarkan nilai validation MAE terendah.
Konfigurasi tersebut kemudian dievaluasi menggunakan test MAE dan
Natal MAE untuk mengetahui kemampuan prediksi model pada data
pengujian. Hasil evaluasi dikaitkan dengan pola trafik pada
Gambar~\ref{fig:pola}. Perubahan trafik yang tidak teratur dan
fluktuasi penggunaan jaringan dapat memengaruhi tingkat kesalahan
prediksi. Oleh karena itu, konfigurasi terbaik dipilih berdasarkan
hasil evaluasi aktual dan kemampuannya dalam mengikuti pola trafik
pada dataset Navigli.
 
\begin{table}[t]
  \centering
  \caption{Uji generalisasi: acuan dan baseline naif di Navigli (horizon 1 jam).}
  \label{tab:acuan-navigli}
  \scriptsize
  \setlength{\tabcolsep}{2.5pt}
  \begin{tabular}{@{}lcccc@{}}
    \toprule
    Model & Val MAE & Test MAE & Test MAPE (\%) & Natal MAE \\
    \midrule
    Naif: nilai terakhir & 324,8 & 272,0 & 11,0 & 177,7 \\
    Naif: kemarin        & 312,0 & 342,3 & 13,1 & 273,2 \\
    Naif: minggu lalu    & 225,7 & 471,4 & 18,0 & 1107,3 \\
    \midrule
    ANN  & $166{,}6 \pm 5{,}3$ & $222{,}3 \pm 27{,}4$ & $8{,}9 \pm 1{,}1$ & $664{,}5 \pm 190{,}8$ \\
    RNN  & $\mathbf{147{,}3 \pm 4{,}9}$ & $193{,}0 \pm 9{,}7$ & $7{,}8 \pm 0{,}3$ & $534{,}7 \pm 148{,}9$ \\
    LSTM & $164{,}3 \pm 9{,}3$ & $247{,}3 \pm 54{,}3$ & $10{,}0 \pm 2{,}7$ & $950{,}8 \pm 110{,}9$ \\
    GRU  & $149{,}0 \pm 6{,}0$ & $\mathbf{190{,}8 \pm 6{,}8}$ & $\mathbf{7{,}8 \pm 0{,}2}$ & $\mathbf{496{,}4 \pm 50{,}9}$ \\
    \bottomrule
  \end{tabular}
\end{table}
 
\begin{figure}[t]
  \centering
  \includegraphics[width=\linewidth]{D_pola_lokasi.png}
  \caption{Pola trafik di lokasi kelompok dibandingkan Duomo.}
  \label{fig:pola}
\end{figure}
 
% ---------------------------------------------------------------------
\section{Pembahasan}
\textbf{Mengapa model rekuren unggul atas ANN.} ANN meratakan seluruh jendela menjadi satu vektor masukan, sehingga jumlah parameternya tumbuh bersama $W$ (6.497 parameter pada $W=24$, 38.753 pada $W=168$) sementara data latih hanya empat minggu. Hal ini sejalan dengan memburuknya ANN pada jendela panjang (A1). Model rekuren memakai bobot yang sama di setiap langkah waktu, sehingga jumlah parameternya tidak bergantung pada $W$ (RNN tetap 5.305 parameter) dan hasilnya hampir tidak berubah terhadap $W$. Kebutuhan akan ``memori panjang'' dengan demikian tampaknya tidak menjadi faktor pembeda pada horizon 1 jam di data ini.
 
\textbf{Perbedaan model versus perbedaan \textit{seed}.} Di Duomo, RNN ($229{,}6 \pm 5{,}1$) dan GRU ($237{,}2 \pm 9{,}3$) hanya berselisih $7{,}6$, lebih kecil daripada simpangan baku gabungannya, sehingga urutan keduanya tidak dapat dianggap pasti. Dengan hanya tiga \textit{seed}, kesimpulan yang kuat hanya berlaku untuk selisih besar, misalnya RNN terhadap ANN ($84{,}7$) atau GRU/RNN terhadap LSTM yang simpangan bakunya besar ($25{,}4$). Di Navigli situasinya serupa: GRU dan RNN setara.
 
\textbf{Hyperparameter paling berpengaruh.} \textit{Learning rate} berpengaruh paling besar: dari $10^{-4}$ ke $10^{-2}$, GRU turun dari $356{,}3$ menjadi $215{,}7$, dan pada $10^{-4}$ seluruh run mencapai batas 300 epoch tanpa konvergen. Jumlah lapisan juga berpengaruh, tetapi merugikan: LSTM sangat memburuk pada tiga lapisan, kemungkinan karena model lebih sulit dilatih pada data yang sedikit. Panjang jendela dan jumlah lapisan hampir tidak berpengaruh pada RNN. \textit{Dropout} 0,5 merugikan semua model; model-model ini berukuran kecil (sekitar $10^3$--$10^4$ parameter), sehingga regularisasi kuat lebih mengurangi kapasitas daripada mencegah \textit{overfitting}. \textit{Gradient clipping} praktis tidak mengubah galat rata-rata, tetapi mengurangi ketidakstabilan ANN pada $\textit{learning rate}$ tinggi.
 
\textbf{Fitur.} Fitur panggilan dan SMS membantu model rekuren (RNN $254{,}1 \to 227{,}6$) tetapi tidak membantu ANN, yang paling baik dengan fitur \texttt{net} saja. Dugaan kami, ANN dengan masukan lebih banyak ($24 \times 7$ nilai) lebih mudah \textit{overfit}; dugaan ini belum diuji secara langsung.
 
\textbf{Lokasi dan minggu Natal.} Urutan model berubah dari Duomo ke Navigli (ANN mengungguli LSTM), sehingga kesimpulan dari satu lokasi tidak otomatis berlaku di lokasi lain. Pada minggu Natal, model yang dilatih pada minggu biasa jauh lebih buruk daripada baseline ``nilai terakhir'' di Navigli ($496{,}4$ sampai $950{,}8$ dibandingkan $177{,}7$), kemungkinan karena baseline itu langsung mengikuti tingkat trafik terkini. Ini menunjukkan model belum mampu menangani pola hari libur yang tidak ada di data latih.
 
\textbf{Keterbatasan.} Studi ini memakai satu kota, empat minggu data latih, satu kali pembagian waktu, dan hanya tiga \textit{seed}. Tabel ablasi memakai test MAE hanya untuk perbandingan; pemilihan konfigurasi terbaik dilakukan dengan val MAE.
 
% ---------------------------------------------------------------------
\section{Kesimpulan}
Di Duomo, RNN memperoleh test MAE terendah ($229{,}6 \pm 5{,}1$), $27{,}0\%$ lebih rendah daripada ANN ($314{,}3 \pm 10{,}4$), tetapi selisihnya terhadap GRU ($237{,}2 \pm 9{,}3$) tidak lebih besar daripada variasi antar \textit{seed}. Di Navigli, GRU ($190{,}8 \pm 6{,}8$) dan RNN ($193{,}0 \pm 9{,}7$) setara, sehingga urutan model dapat berubah antar lokasi. \textit{Learning rate} adalah faktor yang paling berpengaruh, sedangkan panjang jendela hampir tidak memengaruhi model rekuren tetapi merugikan ANN. Pada minggu Natal di Navigli, semua model kalah dari baseline ``nilai terakhir''.
\isi Pada dataset Duomo, konfigurasi terbaik menurunkan test MAE ANN dari $314{,}3 \pm 10{,}4$ menjadi $306{,}4 \pm 14{,}9$, sedangkan LSTM menurun dari $273{,}6 \pm 25{,}4$ menjadi
$258{,}9 \pm 25{,}5$. Namun, perbaikan tersebut belum menunjukkan
keunggulan yang meyakinkan dibandingkan variasi antar \textit{seed}.
Pada RNN, test MAE justru meningkat dari $229{,}6 \pm 5{,}1$
menjadi $238{,}6 \pm 17{,}2$, sedangkan GRU hanya membaik sedikit,
dari $237{,}2 \pm 9{,}3$ menjadi $234{,}7 \pm 9{,}9$.
Di Navigli, konfigurasi terbaik menurunkan test MAE ANN dari
$222{,}3 \pm 27{,}4$ menjadi $202{,}2 \pm 28{,}0$ dan LSTM dari
$247{,}3 \pm 54{,}3$ menjadi $233{,}4 \pm 39{,}1$.
Sebaliknya, test MAE RNN meningkat dari $193{,}0 \pm 9{,}7$
menjadi $198{,}3 \pm 11{,}3$, dan GRU meningkat dari
$190{,}8 \pm 6{,}8$ menjadi $200{,}2 \pm 13{,}8$.
Hasil ini menunjukkan bahwa konfigurasi yang dipilih berdasarkan
validation MAE tidak selalu menghasilkan kinerja pengujian yang
lebih baik di lokasi berbeda.
Pada minggu Natal, hasil juga bervariasi. Di Duomo, Natal MAE ANN
dan LSTM membaik, masing-masing dari $674{,}1 \pm 41{,}8$ menjadi
$632{,}1 \pm 57{,}9$ dan dari $586{,}0 \pm 56{,}8$ menjadi
$542{,}2 \pm 60{,}2$. Namun, Natal MAE RNN dan GRU meningkat.
Di Navigli, Natal MAE ANN dan LSTM menurun, sedangkan RNN dan GRU
meningkat. Dengan demikian, penggabungan nilai terbaik dari setiap
faktor tidak otomatis menghasilkan model terbaik. Interaksi antar
faktor dan perbedaan pola trafik antarlokasi perlu diperhitungkan.

\section*{Kontribusi Anggota}
\begin{tabular}{@{}lp{0.62\linewidth}@{}}
\textbf{Model} & \textbf{Hasil eksperimen} \\
  Grace Victoria Simamora & Menyusun dan mengerjakan laporan latex \\
  Justianaa Insyirah & Mencari informasi dan mengerjakan Google Colab \\
  Tantri Nur Sayfi & Mencari informasi dan menjalankan Google Colab \\
\end{tabular}

\section*{Pernyataan Penggunaan AI}
Claude digunakan untuk mencari informasi dan membantu merapikan bahasa laporan. Seluruh angka berasal dari run kelompok sendiri.

\section*{Kode dan Data}
{Kode, notebook beserta output, dan seluruh hasil run tersedia di \url{https://github.com/tantri124400091-bot/dlw-tugas-ra--1-}.}
\bibliographystyle{IEEEtran}
\bibliography{references}

\end{document}
