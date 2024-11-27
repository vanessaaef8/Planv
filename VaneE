import streamlit as st
import pandas as pd
import plotly.graph_objects as go
import numpy as np
import requests
import yfinance as yf
import pygame

# Configuración inicial de la página
st.set_page_config(
    page_title="Planv",
    page_icon="🏡",
    layout="wide"
)

# Inicializa pygame para manejar los sonidos
pygame.mixer.init()

# Función para reproducir aplausos
def reproducir_aplausos():
    sonido = pygame.mixer.Sound("aplausos.wav")  # Asegúrate de tener el archivo de sonido en el directorio
    sonido.play()

# Configuración de Banxico API
BANXICO_API_TOKEN = "c12f3a32914576b3029870226615defce1527efcc49967ebea0a9d6ed14a7c78"
BANXICO_URL = "https://www.banxico.org.mx/SieAPIRest/service/v1/series/{series_id}/datos/oportuno"

def obtener_tasa_cetes(plazo):
    """
    Obtiene la tasa de rendimiento actual de CETES para un plazo dado desde Banxico.
    """
    series_id = {
        28: "SF43936",  # CETES 28 días
        91: "SF43937",  # CETES 91 días
        182: "SF43938",  # CETES 182 días
        364: "SF43939"   # CETES 364 días
    }.get(plazo, "SF43939")

    headers = {"Bmx-Token": BANXICO_API_TOKEN}
    response = requests.get(BANXICO_URL.format(series_id=series_id), headers=headers)
    data = response.json()

    try:
        return float(data["bmx"]["series"][0]["datos"][0]["dato"]) / 100
    except (KeyError, IndexError):
        st.error("Error al obtener tasas de Banxico.")
        return 0.05  # Predeterminado

def obtener_rendimiento_fondo(ticker, años=1):
    """
    Obtiene el rendimiento promedio anualizado de un fondo indexado desde yfinance.
    """
    try:
        data = yf.download(ticker, period=f"{años}y", interval="1d")
        precio_inicial = data['Adj Close'].iloc[0]
        precio_final = data['Adj Close'].iloc[-1]
        rendimiento_total = (precio_final / precio_inicial) - 1
        return (1 + rendimiento_total) ** (1 / años) - 1
    except Exception as e:
        st.error(f"Error al obtener datos de {ticker}: {e}")
        return 0.10  # Predeterminado

def obtener_rendimiento_cripto(cripto_id, días=365):
    """
    Calcula el rendimiento de una criptomoneda en los últimos días desde CoinGecko.
    """
    url = f"https://api.coingecko.com/api/v3/coins/{cripto_id}/market_chart"
    params = {"vs_currency": "usd", "days": días}
    response = requests.get(url, params=params)
    data = response.json()

    try:
        precios = data["prices"]
        precio_inicial = precios[0][1]
        precio_final = precios[-1][1]
        return (precio_final / precio_inicial) - 1
    except KeyError:
        st.error(f"Error al obtener datos de {cripto_id}.")
        return 0.15  # Predeterminado

# Título de la app
st.title("📊 Planifica tu Futuro")
st.subheader("¡Construye tu camino hacia tus metas financieras!")

# Sidebar para navegación entre secciones
menu = st.sidebar.radio(
    "Navega por la app:",
    ["Inicio", "Configurar Metas", "Recomendaciones", "¿Cómo invertir?"]
)

# Función para bienvenida personalizada
if menu == "Inicio":
    st.header("Plan de Inversión")
    st.write("""
        ¡Bienvenido a nuestra aplicación de planificación financiera!
        Aquí podrás configurar tus metas de inversión, 
        elegir en qué fondos y criptomonedas invertir, y proyectar 
        el crecimiento de tus inversiones a lo largo del tiempo.
        
        Para empezar, por favor completa los campos de tu nombre y teléfono.
    """)

    # Solicitar nombre y teléfono
    nombre = st.text_input("Ingresa tu nombre", placeholder="Tu nombre aquí")
    telefono = st.text_input("Ingresa tu teléfono", placeholder="Tu número aquí")
    
    if nombre and telefono:
        st.success(f"¡Hola {nombre}! Nos alegra que estés aquí. ¡Estás un paso más cerca de cumplir tus objetivos!")
        st.image("https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExbmQ0bG03NHpqcjV2cmZmNWM2cDRud2Jyc29scWNlbXllZHExam04dCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/YRuFixSNWFVcXaxpmX/giphy.gif", width=300)
        st.write("¡Vamos a planificar tu futuro financiero!")
        reproducir_aplausos()
    else:
        st.warning("Por favor, ingresa tu nombre y teléfono para continuar con tu planificación.")

# Configurar Metas Financieras (Compra de la Casa)
elif menu == "Configurar Metas":
    st.header("💰 Configurar Metas Financieras")
    st.write("Ingresa los detalles de tu meta de compra de la casa:")
    
    # Monto de la casa deseada (ahora en pesos)
    monto_casa = st.number_input("Monto de la casa deseada (en pesos)", min_value=100_000.0, step=10_000.0)
    plazo_casa = st.slider("Plazo para alcanzar esta meta (en años)", min_value=1, max_value=30, step=1)
    
    if monto_casa > 0 and plazo_casa > 0:
        st.session_state.monto_casa = monto_casa
        st.session_state.plazo_casa = plazo_casa

        if monto_casa < 100_000:
            st.warning("El monto para la casa parece muy bajo. Asegúrate de ingresar un valor realista.")
        if plazo_casa < 2:
            st.error("Un plazo menor a 2 años puede ser muy corto para ahorrar de manera efectiva.")
        
        if st.button("Calcular ahorro anual y metas"):
            st.markdown("### ✨ Resultados del cálculo ✨")
            st.write(f"Tu meta para la casa es de **${monto_casa:,.2f}** en {plazo_casa} años.")

            # Calcular ahorro anual
            ahorro_anual = monto_casa / plazo_casa
            st.write(f"Para alcanzar esta meta, debes ahorrar aproximadamente **${ahorro_anual:,.2f}** al año.")
            st.info("Ahora pasa a la sección **Recomendaciones** para optimizar tus inversiones.")
            st.markdown("---")
    else:
        st.error("Por favor, ingresa valores válidos para continuar.")

# Recomendaciones de inversión
elif menu == "Recomendaciones":
    st.header("💡 Recomendaciones de Inversión")
    if 'monto_casa' not in st.session_state or 'plazo_casa' not in st.session_state:
        st.error("Por favor, configura la meta de compra de la casa antes de recibir recomendaciones.")
    else:
        monto_casa = st.session_state.monto_casa
        plazo_casa = st.session_state.plazo_casa
        st.write(f"**Meta para la casa:** ${monto_casa:,.2f} en {plazo_casa} años.")

        # Selección de perfil de riesgo
        perfil_riesgo = st.selectbox(
            "Selecciona tu perfil de riesgo:",
            ["Conservador", "Moderado", "Agresivo"]
        )

        # Explicación del perfil
        if perfil_riesgo == "Conservador":
            st.write("🔵 **Conservador:** Prefieres inversiones seguras y de bajo riesgo. Ideal para quienes buscan estabilidad en su capital.")
            cetes_pct, fondos_indexados_pct, criptomonedas_pct = 0.7, 0.2, 0.1
        elif perfil_riesgo == "Moderado":
            st.write("🟢 **Moderado:** Combinas seguridad y algo de riesgo para obtener mejores rendimientos.")
            cetes_pct, fondos_indexados_pct, criptomonedas_pct = 0.5, 0.3, 0.2
        else:
            st.write("🔴 **Agresivo:** Buscas maximizar los rendimientos asumiendo mayores riesgos.")
            cetes_pct, fondos_indexados_pct, criptomonedas_pct = 0.3, 0.4, 0.3

        # Selección de instrumentos
        plazo_cetes = st.selectbox("Selecciona el plazo de CETES:", [28, 91, 182, 364])
        tasa_cetes = obtener_tasa_cetes(plazo_cetes)

        fondo_seleccionado = st.selectbox("Selecciona tu fondo indexado:", ["SPY", "QQQ", "EEM", "URTH"])
        tasa_fondos = obtener_rendimiento_fondo(fondo_seleccionado)

        cripto_seleccionada = st.selectbox("Selecciona tu criptomoneda:", ["stablecoins", "ethereum", "bitcoin"])
        cripto_id = {"stablecoins": "tether", "ethereum": "ethereum", "bitcoin": "bitcoin"}[cripto_seleccionada]
        tasa_cripto = obtener_rendimiento_cripto(cripto_id)

        # Distribución de la inversión
        monto_inversion = st.number_input("Monto total a invertir (en pesos)", min_value=1000.0, step=1000.0)
        cetes = monto_inversion * cetes_pct
        fondos_indexados = monto_inversion * fondos_indexados_pct
        criptomonedas = monto_inversion * criptomonedas_pct

        # Calcular montos finales
        monto_final_cetes = cetes * (1 + tasa_cetes) ** plazo_casa
        monto_final_fondos = fondos_indexados * (1 + tasa_fondos) ** plazo_casa
        monto_final_cripto = criptomonedas * (1 + tasa_cripto) ** plazo_casa

        total_inversiones = monto_final_cetes + monto_final_fondos + monto_final_cripto

        # Mostrar resultados
        st.write("### 💰 Resultados de tus inversiones:")
        st.write(f"- **Cetes ({plazo_cetes} días):** ${monto_final_cetes:,.2f}")
        st.write(f"- **Fondos Indexados ({fondo_seleccionado}):** ${monto_final_fondos:,.2f}")
        st.write(f"- **Criptomonedas ({cripto_seleccionada}):** ${monto_final_cripto:,.2f}")
        st.write(f"#### **Total acumulado:** ${total_inversiones:,.2f}")

        # Comparación con el monto de la casa
        if total_inversiones >= monto_casa:
            st.success(f"✅ ¡Felicidades! Puedes alcanzar la meta de tu casa (${monto_casa:,.2f}). 🎉")
        else:
            st.error(f"❌ El total acumulado (${total_inversiones:,.2f}) no es suficiente para alcanzar la meta de tu casa (${monto_casa:,.2f}).")

            # Gráfica de proyección
            st.markdown("### 📊 Proyección de Inversión")
            fig = go.Figure()
            años = list(range(plazo_casa + 1))
            valores_cetes = [cetes * (1 + tasa_cetes) ** año for año in años]
            valores_fondos = [fondos_indexados * (1 + tasa_fondos) ** año for año in años]
            valores_cripto = [criptomonedas * (1 + tasa_cripto) ** año for año in años]
            fig.add_trace(go.Scatter(x=años, y=valores_cetes, name="Cetes", line=dict(color='green')))
            fig.add_trace(go.Scatter(x=años, y=valores_fondos, name="Fondos Indexados", line=dict(color='blue')))
            fig.add_trace(go.Scatter(x=años, y=valores_cripto, name="Criptomonedas", line=dict(color='orange')))
            fig.update_layout(title="Proyección de Inversión", xaxis_title="Años", yaxis_title="Monto Acumulado")
            st.plotly_chart(fig)

            # Cálculos de los valores acumulados de cada instrumento
            valores_cetes = [cetes * (1 + 0.05) ** año for año in años]
            valores_fondos = [fondos_indexados * (1 + tasa_rendimiento) ** año for año in años]
            valores_cripto = [criptomonedas * (1 + rendimiento_cripto) ** año for año in años]

# Cómo Invertir
elif menu == "Cómo Invertir":
    st.header("📚 Cómo Invertir")
    
    st.subheader("1. Cómo invertir en **Fondos Indexados**")
    st.write("""
        Los fondos indexados son una excelente opción para diversificar tus inversiones y 
        seguir el rendimiento de un índice de mercado. Invertir en un fondo indexado te permite 
        tener una exposición a una variedad de activos sin la necesidad de seleccionar individualmente 
        cada acción o bono.
        
        ### Pasos para invertir en fondos indexados:
        1. **Selecciona un bróker**: Plataformas como **[GBM+](https://www.gbm.com.mx/)** o **[Bursanet](https://www.bursanet.com.mx/)** permiten invertir en estos fondos.
        2. **Abre una cuenta**: Regístrate y abre una cuenta en la plataforma seleccionada.
        3. **Busca el fondo**: Puedes invertir en fondos como **SPY**, **QQQ**, **EEM**, o **URTH**, que siguen índices como el S&P 500.
        4. **Deposita dinero**: Transfiere el monto que deseas invertir.
        5. **Compra el fondo**: Selecciona el fondo que más te convenga y realiza la compra.
    """)

    # Agregar los logos de los brókers
    st.image("https://www.gbm.com.mx/assets/img/gbm-logo.svg", width=150, caption="GBM+")
    st.image("https://www.bursanet.com.mx/wp-content/uploads/2018/02/logo.png", width=150, caption="Bursanet")

    st.subheader("2. Cómo invertir en **Criptomonedas**")
    st.write("""
        Las criptomonedas son monedas digitales que funcionan sin un banco central. 
        Bitcoin, Ethereum y stablecoins como USDT son algunas de las opciones más populares.
        
        ### Pasos para invertir en criptomonedas:
        1. **Selecciona una plataforma**: Usa plataformas como **[Binance](https://www.binance.com/)**, **[Coinbase](https://www.coinbase.com/)**, o **[Bitso](https://bitso.com/)** para comprar criptomonedas.
        2. **Crea una cuenta**: Regístrate en la plataforma seleccionada.
        3. **Depósito**: Añade fondos a tu cuenta usando transferencia bancaria o tarjeta.
        4. **Compra criptomonedas**: Compra criptomonedas como **Bitcoin (BTC)** o **Ethereum (ETH)**.
        5. **Almacenamiento**: Guarda tus criptomonedas en una billetera digital.
    """)

    # Agregar los logos de las plataformas de criptomonedas
    st.image("https://www.binance.com/resources/img/logo.svg", width=150, caption="Binance")
    st.image("https://www.coinbase.com/favicon.ico", width=150, caption="Coinbase")
    st.image("https://bitso.com/favicon.ico", width=150, caption="Bitso")

    st.subheader("3. Cómo invertir en **CETES**")
    st.write("""
        Los CETES (Certificados de la Tesorería de la Federación) son instrumentos de deuda emitidos por el gobierno de México. 
        Son considerados de bajo riesgo y tienen plazos de inversión que varían entre 28 y 364 días.
        
        ### Pasos para invertir en CETES:
        1. **Regístrate en CETES Directo**: Ingresa a **[CETES Directo](https://www.cetesdirecto.com)** y abre una cuenta.
        2. **Selecciona un plazo**: Elige entre plazos de 28, 91, 182 o 364 días.
        3. **Realiza la inversión**: Decide cuánto dinero deseas invertir.
        4. **Compra los CETES**: Realiza la compra directamente desde la plataforma de CETES Directo.
        5. **Recibe los rendimientos**: Al final del plazo, recibirás los rendimientos generados.
    """)

    # Agregar el logo de CETES Directo
    st.image("https://www.cetesdirecto.com/images/logos/logo_cetesdirecto.png", width=150, caption="CETES Directo")        
