require 'lib.monetloader'
local imgui = require 'mimgui'
local ffi = require 'ffi'
local sampev = require('lib.samp.events')
local memory = require('memory')
local enc = require('encoding')
local json = require 'json'
local smem = require 'SAMemory'
enc.default = 'CP1251'
local u8 = enc.UTF8
local fa = require 'fAwesome6'
local smem = require 'SAMemory'
local gif = ffi.load(getWorkingDirectory() .. '/lib/libGIF.so')
local gta = ffi.load('GTASA')
local inicfg = require('inicfg')
ffi.cdef[[
    int extractGif(const char* inPath, const char* outDir);
    void _Z12AND_OpenLinkPKc(const char* link);
    bool _ZN4CPad9GetSprintEi(void* this, void* ped, bool bol);
]]
local CFG = getWorkingDirectory() .. "/config/Shell.json"

local showUpdateScreen = false
local function checkFont()
    local fontPath = getWorkingDirectory() .. "/web/fonts/hud.ttf"
    if doesFileExist(fontPath) then
        -- No Android/Moonloader, io.open pode ler o tamanho
        local f = io.open(fontPath, "rb")
        if f then
            local size = f:seek("end")
            f:close()
            -- 124.25 KB é aproximadamente 127232 - 127240 bytes
            if size > 120000 and size < 130000 then
                return true
            end
        end
    end
    return false
end
local savePend = false
local saveTimer = 0
local MDS = (MONET_DPI_SCALE or 0.3) * 0.72
local chatOff = false
local showLangMenu = false
local splashTimer = 0
local showSplash = false
local splashText = "SHELLDER"
local moverJuntos = imgui.new.bool(true)
local moverFontesJuntas = imgui.new.bool(true)
local skinId = 0
local skinEnabled = false
local jumpCirclesEnabled = false
local jumpCircles = {}
local lastOnGround = true
local jumpIni = inicfg.load({
    main = {
        rainbow = true,
        alpha_start = 1,
        alpha_end = 0,
        radius_start = 0.1,
        radius_end = 1,
        duration = 0.5,
    },
    color = { r = 1, g = 0, b = 0.3, a = 1 }
}, "JumpCircles.ini")
lang = {
    [0] = {
        t="HassleHUD Android", g="HUD", c="CORES", cr="EXTRA",
        oh="HUD Original", vals="Mostrar valores nas barras", st2="Estilo 2",
        cspr="Círculo de stamina",
        th="Tamanho do HUD", px="Posição X", py="Posição Y",
        mp="Mover HUD", ma="Arrastar para mover",
        lb="Largura da barra", ab="Altura da barra",
        bx="Barras X", by="Barras Y",
        rd="Barras arredondadas", rdlv="Nível de arredondamento",
        hpe="Mostrar armadura", mfl="Mostrar stamina",
        dx="Dinheiro X", dy="Dinheiro Y", fdin="Tam. fonte dinheiro",
        pa="Fist animado", tpa="Tam. fist",
        ix="Ícone X", iy="Ícone Y", ti="Tam. ícones",
        icx="Ícones X", icy="Ícones Y",
        ma="Mostrar munição", mx="Munição X", my="Munição Y",
        fma="Tam. fonte munição", me="Mostrar estrelas",
        mel="Estrelas em linha", te="Tam. estrelas",
        ox="Offset X", oy="Offset Y", ee="Espaço entre estrelas",
        e2x="Estrelas Estilo 2 X", e2y="Estrelas Estilo 2 Y",
        v="Vida", a="Armadura", s="Stamina",
        d="Dinheiro", e="Estrelas", iv="Ícone vida",
        ia="Ícone armadura", m="Munição", id="Idioma",
        crd="Criador", ver="Versão: 1.0",
        armas="Armas", bs="Barras", fch="Fechar",
        arr="Arrastar", sxy="Sliders XY",
        aa="Arrastar ativo", sa="Sliders ativos",
        sf="Solte para fixar", ds="Dinheiro", rec="Recursos",
        pq="Pequeno", md="Médio", gr="Grande",
        fv="Fonte valores",
        at="Sobre o Mod",
        atv="Versão: 6.0",
        atc="Criador: Shellder",
        at1="HUD inspirado no Hasslehud.",
        at2="Recriado para Android com suporte a múltiplos GIFs.",
        at3="Fist animado compatível com Android.",
        at4="Dois estilos de HUD (Estilo 1 e 2).",
        at5="Seleção de múltiplos GIFs para o fist.",
        at6="Bordas personalizáveis para o fist.",
        att="Como usar os GIFs do fist:",
        att1="1. Coloque os arquivos 'fist1.gif', 'fist2.gif', etc. na pasta",
        att2="   'web/'.",
        att3="2. Selecione o GIF desejado no menu.",
        ft="Tipo do fist",
        fq="Quadrado",
        fc="Circular",
        fb="Tamanho das bordas",
        fg="Selecionar GIF",
        fn="Nenhum fist encontrado",
        cl="Clima", cl_at="Ativar Clima", cl_id="ID do Clima", cl_time="Hora",
        mi="MIRAS", mi_at="Ativar Mira", mi_t="Tipo de Mira", mi_sz="Tamanho", mi_w="Largura", mi_b="Borda", mi_bs="Tamanho Borda",
        rgb="HUD RGB", chat_off="Desativar Chat", skin="Skin", skin_at="Ativar Skin", skin_id="ID da Skin", skin_apply="Aplicar", skin_back="<", skin_next=">", jump="Efeito de Pulo", jump_at="Ativar Efeito",
    },
    [1] = {
        t="HassleHUD Android", g="HUD", c="COLORS", cr="EXTRA",
        oh="Original HUD", vals="Show values on bars", st2="Style 2",
        cspr="Stamina circle",
        th="HUD size", px="Position X", py="Position Y",
        mp="Move HUD", ma="Drag to move",
        lb="Bar width", ab="Bar height",
        bx="Bars X", by="Bars Y",
        rd="Rounded bars", rdlv="Rounding level",
        hpe="Show armor", mfl="Show stamina",
        dx="Money X", dy="Money Y", fdin="Money font size",
        pa="Animated fist", tpa="Fist size",
        ix="Icon X", iy="Icon Y", ti="Icon size",
        icx="Icons X", icy="Icons Y",
        ma="Show ammo", mx="Ammo X", my="Ammo Y",
        fma="Ammo font size", me="Show stars",
        mel="Stars in line", te="Star size",
        ox="Offset X", oy="Offset Y", ee="Star spacing",
        e2x="Stars Style 2 X", e2y="Stars Style 2 Y",
        v="Health", a="Armor", s="Stamina",
        d="Money", e="Stars", iv="Health icon",
        ia="Armor icon", m="Ammo", id="Language",
        crd="Author", ver="Version: 1.0",
        armas="Weapons", bs="Bars", fch="Close",
        arr="Drag", sxy="XY sliders",
        aa="Drag active", sa="Sliders active",
        sf="Release to lock", ds="Money", rec="Features",
        pq="Small", md="Medium", gr="Large",
        fv="Values font",
        at="About",
        atv="Version: 6.0",
        atc="Author: Shellder",
        at1="HUD inspired by PC Hasslehud.",
        at2="Rebuilt for Android with multiple GIF support.",
        at3="Animated fist compatible with Android.",
        at4="Two HUD styles (Style 1 and 2).",
        at5="Multiple GIF selection for the fist.",
        at6="Customizable borders for the fist.",
        att="How to use fist GIFs:",
        att1="1. Place 'fist1.gif', 'fist2.gif', etc. in",
        att2="   'web/'.",
        att3="2. Select the desired GIF in the menu.",
        ft="Fist type",
        fq="Square",
        fc="Circular",
        fb="Border size",
        fg="Select GIF",
        fn="No fist found",
        cl="Weather", cl_at="Enable Weather", cl_id="Weather ID", cl_time="Hour",
        mi="CROSSHAIR", mi_at="Enable Crosshair", mi_t="Crosshair Type", mi_sz="Size", mi_w="Width", mi_b="Border", mi_bs="Border Size",
        rgb="HUD RGB", chat_off="Disable Chat", skin="Skin", skin_at="Enable Skin", skin_id="Skin ID", skin_apply="Apply", skin_back="<", skin_next=">", jump="Jump Effect", jump_at="Enable Effect",
    },
    [2] = {
        t="HassleHUD Android", g="HUD", c="COLORES", cr="EXTRA",
        oh="HUD Original", vals="Mostrar valores en barras", st2="Estilo 2",
        cspr="Círculo de estamina",
        th="Tamaño del HUD", px="Posición X", py="Posición Y",
        mp="Mover HUD", ma="Arrastrar para mover",
        lb="Ancho de barra", ab="Alto de barra",
        bx="Barras X", by="Barras Y",
        rd="Barras redondeadas", rdlv="Nivel redondeo",
        hpe="Mostrar armadura", mfl="Mostrar estamina",
        dx="Dinero X", dy="Dinheiro Y", fdin="Tam. fuente dinero",
        pa="Fist animado", tpa="Tam. fist",
        ix="Icono X", iy="Icono Y", ti="Tam. iconos",
        icx="Iconos X", icy="Iconos Y",
        ma="Mostrar munición", mx="Munición X", my="Munición Y",
        fma="Tam. fuente munición", me="Mostrar estrellas",
        mel="Estrellas en línea", te="Tam. estrellas",
        ox="Offset X", oy="Offset Y", ee="Espacio estrellas",
        e2x="Estrellas Estilo 2 X", e2y="Estrelas Estilo 2 Y",
        v="Vida", a="Armadura", s="Estamina",
        d="Dinero", e="Estrellas", iv="Icono vida",
        ia="Icono armadura", m="Munición", id="Idioma",
        crd="Creador", ver="Versión: 6.0",
        armas="Armas", bs="Barras", fch="Cerrar",
        arr="Arrastrar", sxy="Sliders XY",
        aa="Arrastrar activo", sa="Sliders activos",
        sf="Soltar para fijar", ds="Dinero", rec="Recursos",
        pq="Pequeño", md="Medio", gr="Grande",
        fv="Fuente valores",
        at="Sobre el Mod",
        atv="Versión: 6.0",
        atc="Creador: Shellder",
        at1="HUD inspirado en Hasslehud.",
        at2="Recreado para Android con soporte de GIFs.",
        at3="Fist animado compatible con Android.",
        at4="Dos estilos de HUD (Estilo 1 y 2).",
        at5="Selección de múltiples GIFs para el fist.",
        at6="Bordes personalizáveis para el fist.",
        att="Cómo usar los GIFs del fist:",
        att1="1. Coloque los archivos 'fist1.gif', 'fist2.gif' en",
        att2="   'web/'.",
        att3="2. Seleccione el GIF deseado en el menú.",
        ft="Tipo de fist",
        fq="Cuadrado",
        fc="Circular",
        fb="Tamaño bordes",
        fg="Seleccionar GIF",
        fn="No se encontró fist",
        cl="Clima", cl_at="Activar Clima", cl_id="ID del Clima", cl_time="Hora",
        mi="MIRAS", mi_at="Activar Mira", mi_t="Tipo de Mira", mi_sz="Tamaño", mi_w="Ancho", mi_b="Borde", mi_bs="Tam. Borde",
        rgb="HUD RGB", chat_off="Desativar Chat", skin="Skin", skin_at="Activar Skin", skin_id="ID Skin", skin_apply="Aplicar", skin_back="<", skin_next=">", jump="Efecto Salto", jump_at="Activar Efecto",
    },
    [3] = {
        t="HassleHUD Android", g="HUD", c="ALWAN", cr="IDAFY",
        oh="HUD Al-Asli", vals="Idhar al-qiyam", st2="At-Tiraz 2",
        cspr="Da'irat al-tahamul",
        th="Hajm al-HUD", px="Mawqi' X", py="Mawqi' Y",
        mp="Tahrik al-HUD", ma="Is-hab lil-tahrik",
        lb="Ard al-sharit", ab="Irtifa' al-sharit",
        bx="Sharit X", by="Sharit Y",
        rd="Huwaf muqawwasa", rdlv="Mustawa al-taqwis",
        hpe="Idhar al-dir'", mfl="Idhar al-tahamul",
        dx="Al-mal X", dy="Al-mal Y", fdin="Hajm khat al-mal",
        pa="Fist mutaharrik", tpa="Hajm al-fist",
        ix="Ayquna X", iy="Ayquna Y", ti="Hajm al-ayqunat",
        icx="Ayqunat X", icy="Ayqunat Y",
        ma="Idhar al-dhakhira", mx="Dhakhira X", my="Dhakhira Y",
        fma="Hajm khat al-dhakhira", me="Idhar al-nujum",
        mel="Nujum mutataliya", te="Hajm al-nujum",
        ox="Izaha X", oy="Izaha Y", ee="Masafa bayn al-nujum",
        e2x="Nujum At-Tiraz 2 X", e2y="Nujum At-Tiraz 2 Y",
        v="As-siha", a="Ad-dir'", s="At-tahamul",
        d="Al-mal", e="An-nujum", iv="Ayqunat as-siha",
        ia="Ayqunat ad-dir'", m="Ad-dhakhira", id="Al-lugha",
        crd="Al-mubarmij", ver="Al-isdar: 6.0",
        armas="Al-asliha", bs="Al-ashrita", fch="Ighlaq",
        arr="Sabh", sxy="Munzaliqat XY",
        aa="Al-sabh nashit", sa="Al-munzaliqat nashita",
        sf="Atruk lil-tathbit", ds="Al-mal", rec="Al-mizāt",
        pq="Saghir", md="Mutawassit", gr="Kabir",
        fv="Khat al-qiyam",
        at="Hawl al-mod",
        atv="Al-isdar: 6.0",
        atc="Al-mubarmij: Shellder",
        at1="HUD mustawha min Hasslehud.",
        at2="Tam i'adat tasmihuhu lil-Android.",
        at3="Fist mutaharrik mutawafiq ma'a Android.",
        at4="Tirazan lil-HUD (1 wa 2).",
        at5="Ikhtiyar GIFs muta'addida lil-fist.",
        at6="Huwaf qabila lil-takhsis.",
        att="Kayfiyat istikhdam GIFs:",
        att1="1. Da' malaffat 'fist1.gif' fi",
        att2="   'web/'.",
        att3="2. Ikhtar al-GIF al-matlub min al-qa'ima.",
        ft="Naw' al-fist",
        fq="Murabba'",
        fc="Da'iri",
        fb="Hajm al-huwaf",
        fg="Ikhtar GIF",
        fn="Lam yatim al-'uthur 'ala fist",
        cl="At-taqs", cl_at="Taf'il at-taqs", cl_id="Huwiyat at-taqs", cl_time="As-sa'a",
        mi="AL-NISHAN", mi_at="Taf'il al-nishan", mi_t="Naw' al-nishan", mi_sz="Al-hajm", mi_w="Al-ard", mi_b="Al-haffa", mi_bs="Hajm al-haffa",
        rgb="HUD RGB", chat_off="Ta'til al-chat", skin="As-skin", skin_at="Taf'il as-skin", skin_id="Huwiyat as-skin", skin_apply="Tatbiq", skin_back="<", skin_next=">", jump="Ta'thir al-qafz", jump_at="Taf'il al-ta'thir",
    }
}
local idioma = imgui.new.int(0)
local function T(k) return (lang[idioma[0]] and lang[idioma[0]][k]) or k end
savedColors = {
    hp = {0.35,0.60,0.30,1.0},
    ap = {0.90,0.90,0.90,1.0},
    spr = {0.95,0.60,0.10,1.0},
    d = {1.00,1.00,1.00,1.0},
    e = {1.00,1.00,0.00,1.0},
    ic = {1.00,1.00,1.00,1.0},
    ic2 = {1.00,1.00,1.00,1.0},
    m = {1.00,1.00,1.00,1.0},
}
def = {
    main = {
        oh=false, vals=true, sz=2,
        pa=true, psz=1.5, me=true, mel=false, spx=0,
        te=12, rd=true, rdlv=20, ic=true, cspr=false,
        hpe=false, ma=true,         st2=true, mfl=true,
        fdin=20, fma=16, id=0,
        fistBordeScale=130, fistCircular=false, fistIdx=1,
        weatherTime=12, weatherId=0, weatherEnabled=false, hudRGB=false,
    },
    st1 = { hudX=1770, hudY=300, dx=0,dy=0,bx=0,by=0,mx=0,my=0,ix=0,iy=0,lb=156,ab=10,ee=0,ox=0,oy=0, moverJuntos=true, moverFontesJuntas=true, tiv=25, tia=25, hfx=0, hfy=0, afx=0, afy=0, hfs=15, afs=15 },
    st2 = { hudX=1770, hudY=300, dx=0,dy=0,bx=0,by=0,mx=0,my=0,ix=0,iy=0,lb=156,ab=10,ee=0,ox=0,oy=0,icx=0,icy=0,ti=50,e2x=0,e2y=0, moverJuntos=true, moverFontesJuntas=true, tiv=25, tia=25, hfx=0, hfy=0, afx=0, afy=0, hfs=15, afs=15 },
    clr = {
        hp={0.35,0.60,0.30,1.0}, ap={0.90,0.90,0.90,1.0},
        spr={0.95,0.60,0.10,1.0},
        d={1.00,1.00,1.00,1.0}, e={1.00,1.00,0.00,1.0},
        e2={1.00,1.00,1.00,0.24}, ic={1.00,1.00,1.00,1.0},
        ic2={1.00,1.00,1.00,1.0}, m={1.00,1.00,1.00,1.0},
        fhp={1.00,1.00,1.00,1.0}, fap={1.00,1.00,1.00,1.0},
    }
}
local function loadCfg()
    local d = def
    local f = io.open(CFG, "r")
    if f then
        local ok, s = pcall(json.decode, f:read("*a"))
        f:close()
        if ok and s then
            local function g(k, d) return s[k] ~= nil and s[k] or d end
            return {
        main = {
            oh=g("oh",d.main.oh), vals=g("vals",d.main.vals),
            sz=g("sz",d.main.sz), pa=g("pa",d.main.pa),
            psz=g("psz",d.main.psz), me=g("me",d.main.me),
            mel=g("mel",d.main.mel), spx=g("spx",d.main.spx),
            te=g("te",d.main.te), rd=g("rd",d.main.rd),
            rdlv=g("rdlv",d.main.rdlv), ic=g("ic",d.main.ic),
            cspr=g("cspr",d.main.cspr),
            hpe=g("hpe",d.main.hpe), ma=g("ma",d.main.ma),
            st2=g("st2",d.main.st2), mfl=g("mfl",d.main.mfl),
            fdin=g("fdin",d.main.fdin), fma=g("fma",d.main.fma),
            fv=g("fv",15), id=g("id",d.main.id),
            fistBordeScale=g("fistBordeScale",130),
            fistCircular=g("fistCircular",false),
            fistIdx=g("fistIdx",1),
            weatherTime=g("weatherTime",12), weatherId=g("weatherId",0), weatherEnabled=g("weatherEnabled",false), hudRGB=g("hudRGB",false),
            miraAtivada=g("miraAtivada",false), miraTipo=g("miraTipo",0),
            miraTamanho=g("miraTamanho",10.0), miraLargura=g("miraLargura",2.0),
            miraBordaAtivada=g("miraBordaAtivada",true), miraTamanhoBorda=g("miraTamanhoBorda",1.0),
            miraX=g("miraX",0), miraY=g("miraY",0),
            barS2Rnd=g("barS2Rnd",4),
        },
                st1 = {
                    hudX=g("st1_x",d.st1.hudX), hudY=g("st1_y",d.st1.hudY),
                    dx=g("st1_dx",0),dy=g("st1_dy",0),bx=g("st1_bx",0),by=g("st1_by",0),
                    mx=g("st1_mx",0),my=g("st1_my",0),ix=g("st1_ix",0),iy=g("st1_iy",0),
                    lb=g("st1_lb",156),ab=g("st1_ab",10),ee=g("st1_ee",0),
                    ox=g("st1_ox",0),oy=g("st1_oy",0),
                    tiv=g("st1_tiv",25), tia=g("st1_tia",25),
                    hfx=g("st1_hfx",0), hfy=g("st1_hfy",0),
                    afx=g("st1_afx",0), afy=g("st1_afy",0),
                    hfs=g("st1_hfs",15), afs=g("st1_afs",15),
                },
                st2 = {
                    hudX=g("st2_x",d.st2.hudX), hudY=g("st2_y",d.st2.hudY),
                    dx=g("st2_dx",0),dy=g("st2_dy",0),bx=g("st2_bx",0),by=g("st2_by",0),
                    mx=g("st2_mx",0),my=g("st2_my",0),ix=g("st2_ix",0),iy=g("st2_iy",0),
                    lb=g("st2_lb",156),ab=g("st2_ab",10),ee=g("st2_ee",0),
                    ox=g("st2_ox",0),oy=g("st2_oy",0),
                    icx=g("st2_icx",0),icy=g("st2_icy",0),ti=g("st2_ti",50),
                    e2x=g("st2_e2x",0),e2y=g("st2_e2y",0),
                    tiv=g("st2_tiv",25), tia=g("st2_tia",25),
                    hfx=g("st2_hfx",0), hfy=g("st2_hfy",0),
                    afx=g("st2_afx",0), afy=g("st2_afy",0),
                    hfs=g("st2_hfs",15), afs=g("st2_afs",15),
                },
                clr = {
                    hp={g("hp_r",0.35),g("hp_g",0.60),g("hp_b",0.30),g("hp_a",1.0)},
                    ap={g("ap_r",0.90),g("ap_g",0.90),g("ap_b",0.90),g("ap_a",1.0)},
                    spr={g("spr_r",0.95),g("spr_g",0.60),g("spr_b",0.10),g("spr_a",1.0)},
                    d={g("d_r",1.0),g("d_g",1.0),g("d_b",1.0),g("d_a",1.0)},
                    e={g("e_r",1.0),g("e_g",1.0),g("e_b",0.0),g("e_a",1.0)},
                    e2={g("e2_r",1.0),g("e2_g",1.0),g("e2_b",1.0),g("e2_a",0.24)},
                    ic={g("ic_r",1.0),g("ic_g",1.0),g("ic_b",1.0),g("ic_a",1.0)},
                    ic2={g("ic2_r",1.0),g("ic2_g",1.0),g("ic2_b",1.0),g("ic2_a",1.0)},
                    m={g("m_r",1.0),g("m_g",1.0),g("m_b",1.0),g("m_a",1.0)},
                    mira={g("mira_r",1.0),g("mira_g",1.0),g("mira_b",1.0),g("mira_a",1.0)},
                    bmira={g("bmira_r",0.0),g("bmira_g",0.0),g("bmira_b",0.0),g("bmira_a",1.0)},
                    fhp={g("fhp_r",1.0),g("fhp_g",1.0),g("fhp_b",1.0),g("fhp_a",1.0)},
                    fap={g("fap_r",1.0),g("fap_g",1.0),g("fap_b",1.0),g("fap_a",1.0)},
                }
            }
        end
    end
    return { main=d.main, st1=d.st1, st2=d.st2, clr=d.clr }
end
local cfg = loadCfg()
savedColors = {
    hp = {cfg.clr.hp[1], cfg.clr.hp[2], cfg.clr.hp[3], cfg.clr.hp[4]},
    ap = {cfg.clr.ap[1], cfg.clr.ap[2], cfg.clr.ap[3], cfg.clr.ap[4]},
    spr = {cfg.clr.spr[1], cfg.clr.spr[2], cfg.clr.spr[3], cfg.clr.spr[4]},
    d = {cfg.clr.d[1], cfg.clr.d[2], cfg.clr.d[3], cfg.clr.d[4]},
    e = {cfg.clr.e[1], cfg.clr.e[2], cfg.clr.e[3], cfg.clr.e[4]},
    ic = {cfg.clr.ic[1], cfg.clr.ic[2], cfg.clr.ic[3], cfg.clr.ic[4]},
    ic2 = {cfg.clr.ic2[1], cfg.clr.ic2[2], cfg.clr.ic2[3], cfg.clr.ic2[4]},
    m = {cfg.clr.m[1], cfg.clr.m[2], cfg.clr.m[3], cfg.clr.m[4]},
}
smem.require 'CPlayerInfo'
smem.require 'CPlayerData'
local samemory = require 'SAMemory'
samemory.require 'CPed'
samemory.require 'CPlayerData'
local new = imgui.new
sets = new.bool(false)
curT = new.int(1)
local WDL = nil
local wW, wH = 0, 0
vals = new.bool(cfg.main.vals)
oh = new.bool(cfg.main.oh)
hsz = new.int(cfg.main.sz)
pa = new.bool(cfg.main.pa)
psz = new.float(cfg.main.psz)
me = new.bool(cfg.main.me)
mel = new.bool(cfg.main.mel)
te = new.int(cfg.main.te)
rd = new.bool(cfg.main.rd)
rdlv = new.int(cfg.main.rdlv)
ic = new.bool(cfg.main.ic)
cspr = new.bool(cfg.main.cspr)
mfl = new.bool(cfg.main.mfl)
hpe = new.bool(cfg.main.hpe)
ma = new.bool(cfg.main.ma)
st2 = new.bool(cfg.main.st2)
fdin = new.int(cfg.main.fdin)
fma = new.int(cfg.main.fma)
fv = new.int(cfg.main.fv or 15)
fistBordeScale = new.int(cfg.main.fistBordeScale or 130)
fistCircular = new.bool(cfg.main.fistCircular or false)
fistIdx = new.int(cfg.main.fistIdx or 1)
hudRGB = new.bool(cfg.main.hudRGB or false)
idioma[0] = cfg.main.id or 0
local initSt = cfg.main.st2 and cfg.st2 or cfg.st1
posX = new.int(initSt.hudX)
posY = new.int(initSt.hudY)
dx = new.int(initSt.dx)
dy = new.int(initSt.dy)
bx = new.int(initSt.bx)
by = new.int(initSt.by)
mx = new.int(initSt.mx)
my = new.int(initSt.my)
ix = new.int(initSt.ix)
iy = new.int(initSt.iy)
lb = new.int(initSt.lb)
ab = new.int(initSt.ab)
ee = new.int(initSt.ee)
ox = new.int(initSt.ox)
oy = new.int(initSt.oy)
tiv = new.int(initSt.tiv or 25)
tia = new.int(initSt.tia or 25)
hfx = new.int(initSt.hfx or 0)
hfy = new.int(initSt.hfy or 0)
afx = new.int(initSt.afx or 0)
afy = new.int(initSt.afy or 0)
hfs = new.int(initSt.hfs or 15)
afs = new.int(initSt.afs or 15)
icx = new.int(cfg.st2.icx)
icy = new.int(cfg.st2.icy)
ti = new.int(cfg.st2.ti)
e2x = new.int(cfg.st2.e2x)
e2y = new.int(cfg.st2.e2y)
local gifList = {}
local gifCount = 0
local frames = {}
local outDir = getWorkingDirectory().."/web/.gif"
local GIF = getWorkingDirectory().."/web/fist.gif"
local totalF = 0
local weap = {}
local icns = {}
local starTex = nil
local fistBase = nil
local bordaTex = nil
local bordaCTex = nil
local weatherTime = new.int(cfg.main.weatherTime or 12)
local weatherId = new.int(cfg.main.weatherId or 0)
local weatherEnabled = new.bool(cfg.main.weatherEnabled or false)
local miraAtivada = new.bool(cfg.main.miraAtivada or false)
local miraTipo = new.int(cfg.main.miraTipo or 0)
local miraTamanho = new.float(cfg.main.miraTamanho or 10.0)
local miraLargura = new.float(cfg.main.miraLargura or 2.0)
local miraBordaAtivada = new.bool(cfg.main.miraBordaAtivada or true)
local miraTamanhoBorda = new.float(cfg.main.miraTamanhoBorda or 1.0)
local corMira = new.float[4](cfg.clr.mira[1], cfg.clr.mira[2], cfg.clr.mira[3], cfg.clr.mira[4])
local corBordaMira = new.float[4](cfg.clr.bmira[1], cfg.clr.bmira[2], cfg.clr.bmira[3], cfg.clr.bmira[4])
local miraX = new.int(cfg.main.miraX or 0)
local miraY = new.int(cfg.main.miraY or 0)
local barS2Rnd = new.int(cfg.main.barS2Rnd or 4)
local corFonteHP = new.float[4]((cfg.clr.fhp or {1,1,1,1})[1], (cfg.clr.fhp or {1,1,1,1})[2], (cfg.clr.fhp or {1,1,1,1})[3], (cfg.clr.fhp or {1,1,1,1})[4])
local corFonteAP = new.float[4]((cfg.clr.fap or {1,1,1,1})[1], (cfg.clr.fap or {1,1,1,1})[2], (cfg.clr.fap or {1,1,1,1})[3], (cfg.clr.fap or {1,1,1,1})[4])
local rainbowIdx = 1
local rainbowAlpha = 1.0
local rainbowFade = 1
local rgbTimer = 0
local stamTimer = stamTimer or 0
local stamVal = stamVal or 100
local plyWeps = {}
local lastWepChk = 0
local uiData = {scroll={}}
local clipAmmo = 0
local totalAmmo = 0
local lastWep = 0
local reloadTimer = 0
local isReload = false
local mMode = false
local mSlider = false
local mDrag = false
local mMX0 = 0
local mMY0 = 0
local mPX0 = 0
local mPY0 = 0
local icAnim = {st=0,dur=0.3,prev=0,act=false,psx=0,pex=0,nsx=0,nex=0}
local rgbColor = {r=1.0, g=0.0, b=0.0}
local rgbDirection = 1
local rgbChannel = 1
local clr = {
    hp = new.float[4](cfg.clr.hp[1], cfg.clr.hp[2], cfg.clr.hp[3], cfg.clr.hp[4]),
    ap = new.float[4](cfg.clr.ap[1], cfg.clr.ap[2], cfg.clr.ap[3], cfg.clr.ap[4]),
    spr = new.float[4](cfg.clr.spr[1], cfg.clr.spr[2], cfg.clr.spr[3], cfg.clr.spr[4]),
    d = new.float[4](cfg.clr.d[1], cfg.clr.d[2], cfg.clr.d[3], cfg.clr.d[4]),
    e = new.float[4](cfg.clr.e[1], cfg.clr.e[2], cfg.clr.e[3], cfg.clr.e[4]),
    e2 = new.float[4](cfg.clr.e2[1], cfg.clr.e2[2], cfg.clr.e2[3], cfg.clr.e2[4]),
    ic = new.float[4](cfg.clr.ic[1], cfg.clr.ic[2], cfg.clr.ic[3], cfg.clr.ic[4]),
    ic2 = new.float[4](cfg.clr.ic2[1], cfg.clr.ic2[2], cfg.clr.ic2[3], cfg.clr.ic2[4]),
    m = new.float[4](cfg.clr.m[1], cfg.clr.m[2], cfg.clr.m[3], cfg.clr.m[4]),
}
local function updateRainbowColors()
    if hudRGB[0] then
        local speed = 0.02
        if rgbChannel == 1 then
            rgbColor.r = rgbColor.r + (speed * rgbDirection)
            if rgbColor.r >= 1.0 then
                rgbColor.r = 1.0
                rgbDirection = -1
                rgbChannel = 2
            elseif rgbColor.r <= 0.0 then
                rgbColor.r = 0.0
                rgbDirection = 1
                rgbChannel = 2
            end
        elseif rgbChannel == 2 then
            rgbColor.g = rgbColor.g + (speed * rgbDirection)
            if rgbColor.g >= 1.0 then
                rgbColor.g = 1.0
                rgbDirection = -1
                rgbChannel = 3
            elseif rgbColor.g <= 0.0 then
                rgbColor.g = 0.0
                rgbDirection = 1
                rgbChannel = 3
            end
        elseif rgbChannel == 3 then
            rgbColor.b = rgbColor.b + (speed * rgbDirection)
            if rgbColor.b >= 1.0 then
                rgbColor.b = 1.0
                rgbDirection = -1
                rgbChannel = 1
            elseif rgbColor.b <= 0.0 then
                rgbColor.b = 0.0
                rgbDirection = 1
                rgbChannel = 1
            end
        end
    end
end
local function applyRGBColors()
    if hudRGB[0] then
        clr.hp[0] = rgbColor.r; clr.hp[1] = rgbColor.g; clr.hp[2] = rgbColor.b
        clr.ap[0] = rgbColor.r; clr.ap[1] = rgbColor.g; clr.ap[2] = rgbColor.b
        clr.spr[0] = rgbColor.r; clr.spr[1] = rgbColor.g; clr.spr[2] = rgbColor.b
        clr.d[0] = rgbColor.r; clr.d[1] = rgbColor.g; clr.d[2] = rgbColor.b
        clr.e[0] = rgbColor.r; clr.e[1] = rgbColor.g; clr.e[2] = rgbColor.b
        clr.ic[0] = rgbColor.r; clr.ic[1] = rgbColor.g; clr.ic[2] = rgbColor.b
        clr.ic2[0] = rgbColor.r; clr.ic2[1] = rgbColor.g; clr.ic2[2] = rgbColor.b
        clr.m[0] = rgbColor.r; clr.m[1] = rgbColor.g; clr.m[2] = rgbColor.b
    end
end
local function restoreSavedColors()
    clr.hp[0] = savedColors.hp[1]; clr.hp[1] = savedColors.hp[2]; clr.hp[2] = savedColors.hp[3]; clr.hp[3] = savedColors.hp[4]
    clr.ap[0] = savedColors.ap[1]; clr.ap[1] = savedColors.ap[2]; clr.ap[2] = savedColors.ap[3]; clr.ap[3] = savedColors.ap[4]
    clr.spr[0] = savedColors.spr[1]; clr.spr[1] = savedColors.spr[2]; clr.spr[2] = savedColors.spr[3]; clr.spr[3] = savedColors.spr[4]
    clr.d[0] = savedColors.d[1]; clr.d[1] = savedColors.d[2]; clr.d[2] = savedColors.d[3]; clr.d[3] = savedColors.d[4]
    clr.e[0] = savedColors.e[1]; clr.e[1] = savedColors.e[2]; clr.e[2] = savedColors.e[3]; clr.e[3] = savedColors.e[4]
    clr.ic[0] = savedColors.ic[1]; clr.ic[1] = savedColors.ic[2]; clr.ic[2] = savedColors.ic[3]; clr.ic[3] = savedColors.ic[4]
    clr.ic2[0] = savedColors.ic2[1]; clr.ic2[1] = savedColors.ic2[2]; clr.ic2[2] = savedColors.ic2[3]; clr.ic2[3] = savedColors.ic2[4]
    clr.m[0] = savedColors.m[1]; clr.m[1] = savedColors.m[2]; clr.m[2] = savedColors.m[3]; clr.m[3] = savedColors.m[4]
end
local function saveCurrentColors()
    savedColors.hp = {clr.hp[0], clr.hp[1], clr.hp[2], clr.hp[3]}
    savedColors.ap = {clr.ap[0], clr.ap[1], clr.ap[2], clr.ap[3]}
    savedColors.spr = {clr.spr[0], clr.spr[1], clr.spr[2], clr.spr[3]}
    savedColors.d = {clr.d[0], clr.d[1], clr.d[2], clr.d[3]}
    savedColors.e = {clr.e[0], clr.e[1], clr.e[2], clr.e[3]}
    savedColors.ic = {clr.ic[0], clr.ic[1], clr.ic[2], clr.ic[3]}
    savedColors.ic2 = {clr.ic2[0], clr.ic2[1], clr.ic2[2], clr.ic2[3]}
    savedColors.m = {clr.m[0], clr.m[1], clr.m[2], clr.m[3]}
end
local function saveStyle(isS2)
    local sc = isS2 and cfg.st2 or cfg.st1
    sc.hudX = posX[0]; sc.hudY = posY[0]
    sc.dx = dx[0]; sc.dy = dy[0]
    sc.bx = bx[0]; sc.by = by[0]
    sc.mx = mx[0]; sc.my = my[0]
    if not isS2 then
        sc.ix = ix[0]; sc.iy = iy[0]
    end
    sc.lb = lb[0]; sc.ab = ab[0]
    sc.ee = ee[0]
    if not isS2 then
        sc.ox = ox[0]; sc.oy = oy[0]
    end
    sc.sz = hsz[0]
    sc.moverJuntos = moverJuntos[0]
    sc.moverFontesJuntas = moverFontesJuntas[0]
    sc.tiv = tiv[0]; sc.tia = tia[0]
    sc.hfx = hfx[0]; sc.hpy = hfy[0]
    sc.afx = afx[0]; sc.afy = afy[0]
    sc.hfs = hfs[0]; sc.afs = afs[0]
    if isS2 then
        sc.icx = icx[0]; sc.icy = icy[0]; sc.ti = ti[0]
        sc.e2x = e2x[0]; sc.e2y = e2y[0]
    end
end
local function loadStyle(isS2)
    local sc = isS2 and cfg.st2 or cfg.st1
    posX[0] = sc.hudX; posY[0] = sc.hudY
    dx[0] = sc.dx; dy[0] = sc.dy
    bx[0] = sc.bx; by[0] = sc.by
    mx[0] = sc.mx; my[0] = sc.my
    ix[0] = sc.ix; iy[0] = sc.iy
    lb[0] = sc.lb; ab[0] = sc.ab
    ee[0] = sc.ee
    ox[0] = sc.ox; oy[0] = sc.oy
    if sc.sz then hsz[0] = sc.sz end
    if sc.moverJuntos ~= nil then moverJuntos[0] = sc.moverJuntos end
    if sc.moverFontesJuntas ~= nil then moverFontesJuntas[0] = sc.moverFontesJuntas end
    tiv[0] = sc.tiv or 25; tia[0] = sc.tia or 25
    hfx[0] = sc.hfx or 0; hfy[0] = sc.hfy or 0
    afx[0] = sc.afx or 0; afy[0] = sc.afy or 0
    hfs[0] = sc.hfs or 15; afs[0] = sc.afs or 15
    if isS2 then
        icx[0] = sc.icx; icy[0] = sc.icy; ti[0] = sc.ti
        e2x[0] = sc.e2x; e2y[0] = sc.e2y
    end
end
local function saveCfg()
    local d = getWorkingDirectory().."/config"
    if not doesDirectoryExist(d) then createDirectory(d) end
    local s1 = cfg.st1; local s2 = cfg.st2; local c = cfg.clr
    local data = {
        oh = cfg.main.oh, vals = cfg.main.vals,
        sz = cfg.main.sz, pa = cfg.main.pa,
        psz = cfg.main.psz, me = cfg.main.me,
        mel = cfg.main.mel, spx = cfg.main.spx,
        te = cfg.main.te, rd = cfg.main.rd, rdlv = cfg.main.rdlv,
        ic = cfg.main.ic, cspr = cfg.main.cspr,
        hpe = cfg.main.hpe, ma = cfg.main.ma,
        st2 = cfg.main.st2, mfl = cfg.main.mfl,
        fdin = cfg.main.fdin, fma = cfg.main.fma,
        fv = cfg.main.fv, id = cfg.main.id,
        fistBordeScale = cfg.main.fistBordeScale,
        fistCircular = cfg.main.fistCircular,
        fistIdx = cfg.main.fistIdx,
        weatherTime = cfg.main.weatherTime, weatherId = cfg.main.weatherId, weatherEnabled = cfg.main.weatherEnabled, hudRGB = cfg.main.hudRGB,
        miraAtivada = cfg.main.miraAtivada, miraTipo = cfg.main.miraTipo,
        miraTamanho = cfg.main.miraTamanho, miraLargura = cfg.main.miraLargura,
        miraBordaAtivada = cfg.main.miraBordaAtivada, miraTamanhoBorda = cfg.main.miraTamanhoBorda,
        miraX = cfg.main.miraX, miraY = cfg.main.miraY,
        barS2Rnd = cfg.main.barS2Rnd,
        st1_x = s1.hudX, st1_y = s1.hudY,
        st1_dx = s1.dx, st1_dy = s1.dy,
        st1_bx = s1.bx, st1_by = s1.by,
        st1_mx = s1.mx, st1_my = s1.my,
        st1_ix = s1.ix, st1_iy = s1.iy,
        st1_lb = s1.lb, st1_ab = s1.ab,
        st1_ee = s1.ee, st1_ox = s1.ox, st1_oy = s1.oy,
        st1_sz = s1.sz,
        st1_tiv = s1.tiv, st1_tia = s1.tia,
        st1_hfx = s1.hfx, st1_hfy = s1.hfy,
        st1_afx = s1.afx, st1_afy = s1.afy,
        st1_hfs = s1.hfs, st1_afs = s1.afs,
        st2_x = s2.hudX, st2_y = s2.hudY,
        st2_dx = s2.dx, st2_dy = s2.dy,
        st2_bx = s2.bx, st2_by = s2.by,
        st2_mx = s2.mx, st2_my = s2.my,
        st2_lb = s2.lb, st2_ab = s2.ab,
        st2_ee = s2.ee,
        st2_icx = s2.icx, st2_icy = s2.icy, st2_ti = s2.ti,
        st2_e2x = s2.e2x, st2_e2y = s2.e2y,
        st2_sz = s2.sz,
        st2_tiv = s2.tiv, st2_tia = s2.tia,
        st2_hfx = s2.hfx, st2_hfy = s2.hfy,
        st2_afx = s2.afx, st2_afy = s2.afy,
        st2_hfs = s2.hfs, st2_afs = s2.afs,
        hp_r = c.hp[1], hp_g = c.hp[2], hp_b = c.hp[3], hp_a = c.hp[4],
        ap_r = c.ap[1], ap_g = c.ap[2], ap_b = c.ap[3], ap_a = c.ap[4],
        spr_r = c.spr[1], spr_g = c.spr[2], spr_b = c.spr[3], spr_a = c.spr[4],
        d_r = c.d[1], d_g = c.d[2], d_b = c.d[3], d_a = c.d[4],
        e_r = c.e[1], e_g = c.e[2], e_b = c.e[3], e_a = c.e[4],
        e2_r = c.e2[1], e2_g = c.e2[2], e2_b = c.e2[3], e2_a = c.e2[4],
        ic_r = c.ic[1], ic_g = c.ic[2], ic_b = c.ic[3], ic_a = c.ic[4],
        ic2_r = c.ic2[1], ic2_g = c.ic2[2], ic2_b = c.ic2[3], ic2_a = c.ic2[4],
        m_r = c.m[1], m_g = c.m[2], m_b = c.m[3], m_a = c.m[4],
        mira_r = c.mira[1], mira_g = c.mira[2], mira_b = c.mira[3], mira_a = c.mira[4],
        bmira_r = c.bmira[1], bmira_g = c.bmira[2], bmira_b = c.bmira[3], bmira_a = c.bmira[4],
        fhp_r = c.fhp[1], fhp_g = c.fhp[2], fhp_b = c.fhp[3], fhp_a = c.fhp[4],
        fap_r = c.fap[1], fap_g = c.fap[2], fap_b = c.fap[3], fap_a = c.fap[4],
    }
    local f = io.open(CFG, "w")
    if f then f:write(json.encode(data)); f:close() end
    savePend = false; saveTimer = 0
end
local function qSave()
    saveStyle(st2[0])
    cfg.main.oh = oh[0]; cfg.main.vals = vals[0]
    cfg.main.sz = hsz[0]; cfg.main.pa = pa[0]
    cfg.main.psz = psz[0]; cfg.main.me = me[0]
    cfg.main.mel = mel[0]; cfg.main.te = te[0]
    cfg.main.rd = rd[0]; cfg.main.rdlv = rdlv[0]
    cfg.main.ic = ic[0]; cfg.main.cspr = cspr[0]
    cfg.main.mfl = mfl[0]; cfg.main.hpe = hpe[0]
    cfg.main.ma = ma[0]; cfg.main.st2 = st2[0]
    cfg.main.fdin = fdin[0]; cfg.main.fma = fma[0]
    cfg.main.fv = fv[0]; cfg.main.id = idioma[0]
    cfg.main.fistBordeScale = fistBordeScale[0]
    cfg.main.fistCircular = fistCircular[0]
    cfg.main.fistIdx = fistIdx[0]
    cfg.main.weatherTime = weatherTime[0]; cfg.main.weatherId = weatherId[0]; cfg.main.weatherEnabled = weatherEnabled[0]; cfg.main.hudRGB = hudRGB[0]
    cfg.main.miraAtivada = miraAtivada[0]; cfg.main.miraTipo = miraTipo[0]
    cfg.main.miraTamanho = miraTamanho[0]; cfg.main.miraLargura = miraLargura[0]
    cfg.main.miraBordaAtivada = miraBordaAtivada[0]; cfg.main.miraTamanhoBorda = miraTamanhoBorda[0]
    cfg.main.miraX = miraX[0]; cfg.main.miraY = miraY[0]
    cfg.main.barS2Rnd = barS2Rnd[0]
    if not hudRGB[0] then
        saveCurrentColors()
        cfg.clr.hp = {clr.hp[0], clr.hp[1], clr.hp[2], clr.hp[3]}
        cfg.clr.ap = {clr.ap[0], clr.ap[1], clr.ap[2], clr.ap[3]}
        cfg.clr.spr = {clr.spr[0], clr.spr[1], clr.spr[2], clr.spr[3]}
        cfg.clr.d = {clr.d[0], clr.d[1], clr.d[2], clr.d[3]}
        cfg.clr.e = {clr.e[0], clr.e[1], clr.e[2], clr.e[3]}
        cfg.clr.ic = {clr.ic[0], clr.ic[1], clr.ic[2], clr.ic[3]}
        cfg.clr.ic2 = {clr.ic2[0], clr.ic2[1], clr.ic2[2], clr.ic2[3]}
        cfg.clr.m = {clr.m[0], clr.m[1], clr.m[2], clr.m[3]}
    end
    cfg.clr.mira = {corMira[0], corMira[1], corMira[2], corMira[3]}
    cfg.clr.bmira = {corBordaMira[0], corBordaMira[1], corBordaMira[2], corBordaMira[3]}
    cfg.clr.fhp = {corFonteHP[0], corFonteHP[1], corFonteHP[2], corFonteHP[3]}
    cfg.clr.fap = {corFonteAP[0], corFonteAP[1], corFonteAP[2], corFonteAP[3]}
    savePend = true; saveTimer = 0
end
local function eOutBack(t)
    local c1=1.70158; local c3=c1+1
    return 1 + c3*(t-1)^3 + c1*(t-1)^2
end
local function getClip()
    local ptr = getCharPointer(PLAYER_PED)
    local slot = getWeapontypeSlot(getCurrentCharWeapon(PLAYER_PED))
    return memory.getuint32(ptr + 0x5A0 + slot*0x1C + 0x0C)
end
local function sprintbreath(valor)
    local pedptr = getCharPointer(PLAYER_PED)
    if pedptr == 0 then return end
    local ped = ffi.cast('CPed*', pedptr)
    if ped.pPlayerData == nil then return end
    local playerData = ffi.cast('CPlayerData*', ped.pPlayerData)
    playerData.fSprintEnergy = valor / 100 * 8.0
end
local function staminaped()
    stamTimer = stamTimer + 1
    if isCharDead(PLAYER_PED) then
        stamVal = 100
        stamTimer = 0
        sprintbreath(100)
        return
    end
    if not isCharOnFoot(PLAYER_PED) then
        if stamTimer > 65 then
            stamVal = math.min(100, stamVal + 1)
            stamTimer = 0
            sprintbreath(stamVal)
        end
        return
    end
    local run = isCharPlayingAnim(PLAYER_PED, "sprint_panic") or
                isCharPlayingAnim(PLAYER_PED, "SPRINT_CIVI") or
                isCharPlayingAnim(PLAYER_PED, "woman_runpanic")
    if run then
        if stamTimer > 37 then
            stamVal = math.max(0, stamVal - 1)
            stamTimer = 0
            sprintbreath(stamVal)
        end
    else
        if stamTimer > 65 and stamVal < 100 then
            stamVal = math.min(100, stamVal + 1)
            stamTimer = 0
            sprintbreath(stamVal)
        end
    end
end
local function updPlyWeps()
    plyWeps = {0}
    for id=1,46 do
        if getAmmoInCharWeapon(PLAYER_PED,id) > 0 then table.insert(plyWeps,id) end
    end
end
local function nextWep()
    if os.clock() - lastWepChk > 1.0 then updPlyWeps(); lastWepChk = os.clock() end
    local cur = getCurrentCharWeapon(PLAYER_PED); local idx = 1
    for i,w in ipairs(plyWeps) do if w == cur then idx = i; break end end
    setCurrentCharWeapon(PLAYER_PED, plyWeps[(idx % #plyWeps) + 1])
end
local function carregarGifs()
    gifList = {}
    gifCount = 0
    local p0 = getWorkingDirectory().."/web/fist.gif"
    if doesFileExist(p0) then
        gifList[1] = p0
        gifCount = 1
    end
    local i = 1
    while i <= 99 do
        local p = getWorkingDirectory().."/web/fist"..i..".gif"
        if doesFileExist(p) then
            gifList[#gifList+1] = p
            gifCount = gifCount + 1
            i = i + 1
        else
            break
        end
    end
    if fistIdx[0] > gifCount and gifCount > 0 then
        fistIdx[0] = gifCount
    end
end
local function carregarFramesGif(idx)
    frames = {}
    totalF = 0
    local p = gifList[idx]
    if p and doesFileExist(p) then
        totalF = gif.extractGif(p, outDir)
        for i = 0, totalF - 1 do
            frames[i] = renderLoadTextureFromFile(
                string.format("%s/web/.gif/%d.bmp", getWorkingDirectory(), i)
            )
        end
    end
end
function loadAll()
    carregarGifs()
    if gifCount > 0 then
        if fistIdx[0] <= 0 then fistIdx[0] = 1 end
        carregarFramesGif(fistIdx[0])
    elseif doesFileExist(GIF) then
        gifList[1] = GIF
        gifCount = 1
        carregarFramesGif(1)
        fistIdx[0] = 1
    end
    local sp = getWorkingDirectory().."/web/estrella.png"
    if doesFileExist(sp) then starTex = imgui.CreateTextureFromFile(sp) end
    local bp = getWorkingDirectory().."/web/borda1.png"
    if doesFileExist(bp) then bordaTex = imgui.CreateTextureFromFile(bp) end
    local bcp = getWorkingDirectory().."/web/borda2.png"
    if doesFileExist(bcp) then bordaCTex = imgui.CreateTextureFromFile(bcp) end
end
local function u32(c) return imgui.ColorConvertFloat4ToU32(c) end
local function V4(r,g,b,a) return imgui.ImVec4(r,g,b,a) end
local function V2(x,y) return imgui.ImVec2(x,y) end
    local function getRainbowColor(timeOffset)
        local t = (os.clock() + timeOffset) * 0.5
        local r = math.sin(t * 2 * math.pi) * 0.5 + 0.5
        local g = math.sin((t + 0.33) * 2 * math.pi) * 0.5 + 0.5
        local b = math.sin((t + 0.66) * 2 * math.pi) * 0.5 + 0.5
        return imgui.ImVec4(r, g, b, 1)
    end
    local function drawRainbowText(DL, pos, text)
        local currentX = pos.x
        for i = 1, #text do
            local char = text:sub(i, i)
            local color = getRainbowColor(i * 0.1)
            local charSize = imgui.CalcTextSize(char)
            DL:AddText(imgui.ImVec2(currentX, pos.y), imgui.ColorConvertFloat4ToU32(color), char)
            currentX = currentX + charSize.x
        end
    end
local function intSep(n)
    n = tostring(n)
    if n and #n > 3 then
        local b,e = ("%d"):format(n):gsub("^%-","")
        local c = b:reverse():gsub("%d%d%d","%1.")
        local d = c:reverse():gsub("^%.","")
        return n:gsub(n,(e==1 and "-" or "")..d)
    end
    return n
end
local function BScroll(id, sz)
    local io = imgui.GetIO(); local mp = io.MousePos
    if not uiData.scroll[id] then
        uiData.scroll[id] = {s=0,act=false,ly=0,vel=0,allow=true}
    end
    local d = uiData.scroll[id]
    imgui.BeginChild(id,sz,false,imgui.WindowFlags.NoScrollbar + imgui.WindowFlags.NoScrollWithMouse + imgui.WindowFlags.NoMove)
    if d.allow and imgui.IsWindowHovered(imgui.HoveredFlags.ChildWindows) and io.MouseDown[0] and not imgui.IsAnyItemActive() and not d.act then
        d.act = true; d.ly = mp.y; d.vel = 0
    end
    if not io.MouseDown[0] and d.act then d.act = false end
    local ms = imgui.GetScrollMaxY()
    if d.allow and d.act then
        dy = d.ly - mp.y; d.ly = mp.y; d.s = d.s + dy; d.vel = dy
    elseif math.abs(d.vel) > 0.1 then
        d.s = d.s + d.vel; d.vel = d.vel * 0.9
    else d.vel = 0 end
    if d.s < 0 then d.s = 0 end
    if ms < d.s then d.s = ms end
    imgui.SetScrollY(d.s); d.allow = true
end
local function hit(x,y,w,h)
    local mx,my = imgui.GetMousePos().x, imgui.GetMousePos().y
    return mx >= x and mx <= x + w and my >= y and my <= y + h
end
local function setPlayerSkin(skin)
    local _, id = sampGetPlayerIdByCharHandle(PLAYER_PED)
    local BS = raknetNewBitStream()
    raknetBitStreamWriteInt32(BS, id)
    raknetBitStreamWriteInt32(BS, skin)
    raknetEmulRpcReceiveBitStream(153, BS)
    raknetDeleteBitStream(BS)
end
local function drawCircleIn3d(x, y, z, radius, color, width, polygons)
    local step = math.floor(360 / (polygons or 36))
    local sX_old, sY_old
    for angle = 0, 360, step do
        local _, sX, sY, sZ, _, _ = convert3DCoordsToScreenEx(radius * math.cos(math.rad(angle)) + x , radius * math.sin(math.rad(angle)) + y , z)
        if sZ > 1 then
            if sX_old and sY_old then
                renderDrawLine(sX, sY, sX_old, sY_old, width, color)
            end
            sX_old, sY_old = sX, sY
        end
    end
end
local function updateJumpCircles()
    if not jumpCirclesEnabled then return end
    local onGround = not isCharInAir(PLAYER_PED)
    if onGround and not lastOnGround then
        table.insert(jumpCircles, { startTime = os.clock() })
    end
    lastOnGround = onGround
    local x, y, z = getCharCoordinates(PLAYER_PED)
    local function lerp(startVal, endVal, t)
        return startVal + (endVal - startVal) * t
    end
    for i, circle in ipairs(jumpCircles) do
        local elapsed = os.clock() - circle.startTime
        local duration = jumpIni.main.duration
        local t = math.min(elapsed / duration, 1)
        local radius = lerp(jumpIni.main.radius_start, jumpIni.main.radius_end, t)
        local alpha = lerp(jumpIni.main.alpha_start, jumpIni.main.alpha_end, t)
        local r, g, b
        if jumpIni.main.rainbow then
            local time = os.clock()
            r = math.floor(math.sin(time * 10) * 127 + 128)
            g = math.floor(math.sin(time * 10 + 2) * 127 + 128)
            b = math.floor(math.sin(time * 10 + 4) * 127 + 128)
        else
            r = jumpIni.color.r * 255
            g = jumpIni.color.g * 255
            b = jumpIni.color.b * 255
        end
        local a = alpha * 255
        local color = bit.bor(bit.lshift(math.floor(a), 24), bit.lshift(math.floor(b), 16), bit.lshift(math.floor(g), 8), math.floor(r))
        drawCircleIn3d(x, y, z - 0.7, radius, color, 3, 36)
        if t >= 1 then
            table.remove(jumpCircles, i)
        end
    end
end
local fHud = nil
local fDin = nil
local fMun = nil
local fVal = nil
local ACENT = V4(0.65,0.65,0.65,1.00)
local ACENT_D = V4(0.30,0.30,0.30,0.18)
local ACENT_M = V4(0.35,0.35,0.35,0.55)
local TXT_M = V4(0.55,0.55,0.55,1.00)
function sampev.onServerMessage(color, text)
    if chatOff then return false end
end
imgui.OnInitialize(function()
    imgui.SwitchContext()
    imgui.GetStyle().AntiAliasedFill = false
    imgui.GetIO().IniFilename = nil
    local res = getWorkingDirectory() .. "/web"
    -- removido carregamento das armas 0 a 46.png
    icns = {}
    for n, d in pairs(icons85) do
        icns[n] = imgui.CreateTextureFromFileInMemory(new('const char*', d), #d)
    end
    starTex = imgui.CreateTextureFromFile(res .. "/estrella.png")
    -- fistBase removido
    local fIO = imgui.GetIO().Fonts
    fIO:Clear()
    local szUI = math.floor(18 * MDS)
    local szVal = math.floor(15 * MDS)
    local szDin = math.floor(20 * MDS)
    local szMun = math.floor(16 * MDS)
    local fPath = getWorkingDirectory() .. "/web/fonts/hud.ttf"
    local fCfg = imgui.ImFontConfig()
    fCfg.MergeMode = true
    fCfg.PixelSnapH = true
    local fRange = new.ImWchar[3](fa.min_range, fa.max_range, 0)
    if doesFileExist(fPath) then
        fHud = fIO:AddFontFromFileTTF(fPath, szUI)
        fIO:AddFontFromMemoryCompressedBase85TTF(fa.get_font_data_base85('solid'), szUI, fCfg, fRange)
        fDin = fIO:AddFontFromFileTTF(fPath, szDin)
        fMun = fIO:AddFontFromFileTTF(fPath, szMun)
        fVal = fIO:AddFontFromFileTTF(fPath, szVal)
    else
        fHud = fIO:AddFontDefault()
        fDin = fHud
        fMun = fHud
        fVal = fHud
    end
    local st = imgui.GetStyle()
    local col = imgui.Col
    local V4i = imgui.ImVec4
    local V2i = imgui.ImVec2
    st.WindowPadding = V2i(10*MDS, 10*MDS)
    st.WindowRounding = 14
    st.ChildRounding = 10
    st.FramePadding = V2i(8*MDS, 5*MDS)
    st.FrameRounding = 8
    st.ItemSpacing = V2i(8*MDS, 7*MDS)
    st.ItemInnerSpacing = V2i(6*MDS, 6*MDS)
    st.IndentSpacing = 18*MDS
    st.WindowTitleAlign = V2i(0.5, 0.5)
    st.ButtonTextAlign = V2i(0.5, 0.5)
    st.GrabRounding = 8
    st.ScrollbarRounding = 8
    st.PopupRounding = 10
    st.Colors[col.Text] = V4i(0.92, 0.92, 0.92, 1)
    st.Colors[col.WindowBg] = V4i(0.08, 0.08, 0.08, 0.75)
    st.Colors[col.ChildBg] = V4i(0.10, 0.10, 0.10, 0.75)
    st.Colors[col.PopupBg] = V4i(0.10, 0.10, 0.10, 0.90)
    st.Colors[col.Border] = V4i(0.18, 0.18, 0.18, 1)
    st.Colors[col.BorderShadow] = V4i(0, 0, 0, 0)
    st.Colors[col.FrameBg] = V4i(0.16, 0.16, 0.16, 1)
    st.Colors[col.FrameBgHovered] = V4i(0.20, 0.20, 0.20, 1)
    st.Colors[col.FrameBgActive] = V4i(0.24, 0.24, 0.24, 1)
    st.Colors[col.TitleBg] = V4i(0.08, 0.08, 0.08, 1)
    st.Colors[col.TitleBgActive] = V4i(0.10, 0.10, 0.10, 1)
    st.Colors[col.TitleBgCollapsed] = V4i(0, 0, 0, 0.51)
    st.Colors[col.SliderGrab] = V4i(0.45, 0.45, 0.45, 1)
    st.Colors[col.SliderGrabActive] = V4i(0.60, 0.60, 0.60, 1)
    st.Colors[col.CheckMark] = V4i(0.70, 0.70, 0.70, 1)
    st.Colors[col.Button] = V4i(0.18, 0.18, 0.18, 1)
    st.Colors[col.ButtonHovered] = V4i(0.26, 0.26, 0.26, 1)
    st.Colors[col.ButtonActive] = V4i(0.34, 0.34, 0.34, 1)
    st.Colors[col.Header] = V4i(0.22, 0.22, 0.22, 0.5)
    st.Colors[col.HeaderHovered] = V4i(0.28, 0.28, 0.28, 0.6)
    st.Colors[col.HeaderActive] = V4i(0.34, 0.34, 0.34, 0.7)
    st.Colors[col.Separator] = V4i(0.18, 0.18, 0.18, 1)
    st.Colors[col.TextSelectedBg] = V4i(0.40, 0.40, 0.40, 0.35)
    st.Colors[col.ScrollbarBg] = V4i(0.08, 0.08, 0.08, 0)
    st.Colors[col.ScrollbarGrab] = V4i(0.28, 0.28, 0.28, 1)
    st.Colors[col.ScrollbarGrabHovered] = V4i(0.36, 0.36, 0.36, 1)
    st.Colors[col.ScrollbarGrabActive] = V4i(0.44, 0.44, 0.44, 1)
    loadAll()
end)
local function sprCircle(DL,x,y,r,pol)
    if not cspr[0] or st2[0] then return end
    local st = stamVal; local a1 = 90 - (360/100)*(st/2); local a2 = 90 + (360/100)*(st/2)
    local R,G,B = clr.spr[0],clr.spr[1],clr.spr[2]
    DL:PathClear(); DL:PathArcTo(V2(x,y),r,math.rad(a1),math.rad(a2),pol); DL:PathFillConvex(u32(V4(R,G,B,0.10)))
    DL:PathClear(); DL:PathArcTo(V2(x,y),r,math.rad(a1),math.rad(a2),pol); DL:PathStroke(u32(V4(R,G,B,0.55)),false,3)
    DL:PathClear(); DL:PathArcTo(V2(x,y),r,math.rad(a1),math.rad(a2),pol); DL:PathStroke(u32(V4(R,G,B,1)),false,1)
end
local function drawFistRounded(DL, tex, cx, cy, sz, alfa)
    if not tex then return end
    local r = sz / 2
    local col32 = u32(V4(1,1,1,alfa or 1))
    DL:AddImageRounded(tex, V2(cx-r,cy-r), V2(cx+r,cy+r), V2(0,0), V2(1,1), col32, r, 15)
    if not st2[0] and bordaCTex then
        local bR = sz * (fistBordeScale[0] / 100.0) / 2
        DL:AddImage(bordaCTex, V2(cx-bR,cy-bR), V2(cx+bR,cy+bR), nil, nil, col32)
    end
end
local function drawFistSquare(DL, tex, cx, cy, sz, alfa)
    if not tex then return end
    local r = sz / 2
    local col32 = u32(V4(1,1,1,alfa or 1))
    DL:AddImage(tex, V2(cx-r,cy-r), V2(cx+r,cy+r), nil, nil, col32)
    if not st2[0] and bordaTex then
        local bR = sz * (fistBordeScale[0] / 100.0) / 2
        DL:AddImage(bordaTex, V2(cx-bR,cy-bR), V2(cx+bR,cy+bR), nil, nil, col32)
    end
end
local function drawWeapon(DL, tex, cx, cy, sz, alfa)
    -- Se estiver no Estilo 2, não desenhamos armas por cima do original aqui
    if st2[0] then return end
    if not tex then return end
    local r = sz / 2
    local col32 = u32(V4(1,1,1,alfa or 1))
    DL:AddImage(tex, V2(cx-r,cy-r), V2(cx+r,cy+r), nil, nil, col32)
end
local function wep1(DL,x,y,r,sM,gIdx,wid)
    local base = r * 1.3
    local sz = base + ix[0]
    local wx = x - sz/2
    local wy = y - sz/2
    
    -- Estilo 1: Usa GIF e Bordas (cobre o original)
    if not st2[0] and pa[0] and totalF > 0 and wid == 0 then
        local fw = sz * psz[0]
        local tex = frames[gIdx % totalF]
        if fistCircular[0] then
            drawFistRounded(DL, tex, x, y, fw, 1)
        else
            drawFistSquare(DL, tex, x, y, fw, 1)
        end
    end
    -- No Estilo 2, não desenhamos nada aqui para deixar o Fist original aparecer.
    
    if wx and hit(wx,wy,sz,sz) and imgui.IsMouseClicked(0) then
        if wid > 0 then nextWep() elseif #plyWeps > 1 then setCurrentCharWeapon(PLAYER_PED, plyWeps[2] or 0) end
    end
end
local function wep2(DL,x,y,gIdx,wid)
    updPlyWeps()
    local tot = #plyWeps
    local curIdx = 1
    for i,w in ipairs(plyWeps) do if w == wid then curIdx = i; break end end
    local csz = ti[0]
    local ssz = csz * 0.83
    local sp = ssz * 0.15
    local bX = x + icx[0]
    local yPos = y - 110 + icy[0]
    local now = os.clock()
    local pIdx = curIdx - 1; if pIdx < 1 then pIdx = tot end
    local nIdx = curIdx + 1; if nIdx > tot then nIdx = 1 end
    local pw = plyWeps[pIdx]
    local nw = plyWeps[nIdx]
    local function tgtX(s)
        if tot == 1 then return bX - csz/2
        elseif tot == 2 then
            local sx = bX - (ssz + sp + csz)/2
            return s == 0 and sx or sx + ssz + sp
        else
            local sx = bX - (ssz + sp + csz + sp + ssz)/2
            if s == 0 then return sx
            elseif s == 1 then return sx + ssz + sp
            else return sx + ssz + sp + csz + sp end
        end
    end
    -- Animação de troca de armas removida
    icAnim.act = false
    icAnim.prev = wid
    local function drawIco(w2,px,py,sz,isCur,slot)
        local tex = nil
        -- Apenas Estilo 1 usa GIF
        if not st2[0] and pa[0] and w2 == 0 and totalF > 0 then
            tex = frames[gIdx % totalF]
        end

        -- Se estiver no Estilo 1 e tiver GIF, desenha o GIF e as bordas
        if tex then
            local cx2 = px + sz/2
            local cy2 = py + sz/2
            if fistCircular[0] then
                drawFistRounded(DL, tex, cx2, cy2, sz, isCur and 1.0 or 0.6)
            else
                drawFistSquare(DL, tex, cx2, cy2, sz, isCur and 1.0 or 0.6)
            end
            -- Retornamos para não deixar o jogo desenhar o original por cima no Estilo 1
            return
        end

        -- Se estiver no Estilo 2 ou se não tiver GIF no Estilo 1, não fazemos nada aqui
        -- para que o Fist original do jogo apareça normalmente.
    end
    if tot == 1 then
        drawIco(plyWeps[1], tgtX(0), yPos, csz, true, 1)
    elseif tot == 2 then
        drawIco(plyWeps[1], tgtX(0), yPos, ssz, plyWeps[1] == wid, 0)
        drawIco(plyWeps[2], tgtX(1), yPos, csz, plyWeps[2] == wid, 1)
    else
        drawIco(pw, tgtX(0), yPos, ssz, false, 0)
        drawIco(wid, tgtX(1), yPos, csz, true, 1)
        drawIco(nw, tgtX(2), yPos, ssz, false, 2)
    end
end
local function bars(DL,x,y,r,sM)
    local rnd = rd[0] and rdlv[0] or 0
    local alt = ab[0] * sM
    local l1 = lb[0]
    local l2 = lb[0] - 50
    local ofL = {15,40}
    local spY = 35 * sM
    local iR = 15 * sM
    local hp = getCharHealth(PLAYER_PED)
    local ap = getCharArmour(PLAYER_PED)
    local bXB = x + bx[0]
    local bYB_base = (st2[0] and y + 20 or y)
    local bYB = bYB_base + by[0]
    local r2 = st2[0] and (70 * sM) or r
    local function bar(ax,ay,bx,by,val,mx,R,G,B,cor)
        local A = V2(ax,ay)
        local Bp = V2(bx,by)
        DL:AddRectFilled(A, Bp, u32(V4(R,G,B,0.25)), rnd, rd[0] and cor or 0)
        Bp.x = ax + ((bx - ax) / mx) * val
        DL:AddRectFilled(A, Bp, u32(V4(R,G,B,1)), rnd, rd[0] and cor or 0)
    end
    local function valTxt(txt, cx, cy)
        imgui.PushFont(fVal)
        local sc = fv[0] / 15.0
        imgui.SetWindowFontScale(sc)
        local ts = imgui.CalcTextSize(txt)
        local px = cx - ts.x/2
        local py = cy - ts.y/2
        DL:AddText(V2(px+1, py+1), 0x70000000, txt)
        DL:AddText(V2(px, py), 0xFFFFFFFF, txt)
        imgui.SetWindowFontScale(1.0)
        imgui.PopFont()
    end
    local lS2 = lb[0] * 0.8
    local hS2 = ab[0] * 2.5
    local spYS2 = hS2 + 10 * sM
    if not st2[0] then
        bar(bXB - r - ofL[1] - l1, bYB - alt/2, bXB - r - ofL[1], bYB + alt/2,
            hp, 100, clr.hp[0],clr.hp[1],clr.hp[2],
            imgui.DrawCornerFlags.BotLeft + imgui.DrawCornerFlags.TopRight)
        if ic[0] and icns["heart"] then
            local Ai = V2(bXB - r - ofL[1] - l1 - 25 - iR, bYB - iR)
            DL:AddImage(icns["heart"], Ai, V2(Ai.x + iR*2, Ai.y + iR*2), nil, nil,
                u32(V4(clr.ic[0],clr.ic[1],clr.ic[2],clr.ic[3])))
        end
        if vals[0] then valTxt(string.format("%d HP", math.floor(hp)), bXB - r - ofL[1] - l1/2, bYB) end
        if ap > 0 or hpe[0] then
            bar(bXB - r - ofL[2] - l2, bYB - spY - alt/2, bXB - r - ofL[2], bYB - spY + alt/2,
                ap, 100, clr.ap[0],clr.ap[1],clr.ap[2],
                imgui.DrawCornerFlags.TopLeft + imgui.DrawCornerFlags.BotRight)
            if ic[0] and icns["shield"] then
                local Ai2 = V2(bXB - r - ofL[2] - l2 - 25 - iR, bYB - spY - iR)
                DL:AddImage(icns["shield"], Ai2, V2(Ai2.x + iR*2, Ai2.y + iR*2), nil, nil,
                    u32(V4(clr.ic2[0],clr.ic2[1],clr.ic2[2],clr.ic2[3])))
            end
            if vals[0] then valTxt(string.format("%d AP", math.floor(ap)), bXB - r - ofL[1] - l1/2, bYB - spY) end
        end
        if mfl[0] then
            bar(bXB - r - ofL[2] - l2, bYB + spY - alt/2, bXB - r - ofL[2], bYB + spY + alt/2,
                stamVal, 100, clr.spr[0],clr.spr[1],clr.spr[2],
                imgui.DrawCornerFlags.BotLeft + imgui.DrawCornerFlags.TopRight)
            if ic[0] and icns["stamina"] then
                local Ab = V2(bXB - r - ofL[2] - l2 - 25 - iR, bYB + spY - iR)
                DL:AddImage(icns["stamina"], Ab, V2(Ab.x + iR*2, Ab.y + iR*2), nil, nil,
                    u32(V4(clr.spr[0],clr.spr[1],clr.spr[2],clr.spr[3]))) -- Cor do ícone sincronizada com a barra
            end
            if vals[0] then valTxt(string.format("%d%%", math.floor(stamVal)), bXB - r - ofL[1] - l1/2, bYB + spY) end
        end
    else
        local function barS2(ax, ay, largura, altura, val, mx, R, G, B, tex, corIcone, corFonte, offX, offY, fSz)
            local fundo = V2(ax, ay)
            local fim = V2(ax + largura, ay + altura)
            local preenchimento = (math.min(val, mx) / mx) * largura
            local frente = V2(ax + preenchimento, ay + altura)
            local raio = barS2Rnd[0] * sM
            DL:AddRectFilled(fundo, fim, u32(V4(0.1, 0.1, 0.1, 0.8)), raio)
            if preenchimento > 0 then
                DL:AddRectFilled(fundo, frente, u32(V4(R, G, B, 1.0)), raio)
            end
            if ic[0] and tex then
                local iSz = altura * 1.2
                local iPos = V2(ax - iSz - 5, ay + (altura - iSz)/2)
                DL:AddImage(tex, iPos, V2(iPos.x + iSz, iPos.y + iSz), nil, nil, corIcone and u32(corIcone) or 0xFFFFFFFF)
            end
            if vals[0] then
                imgui.PushFont(fVal)
                local fscale = (fSz or fv[0]) / 13.0
                imgui.SetWindowFontScale(fscale)
                local txt = string.format("%d", math.floor(val))
                local ts = imgui.CalcTextSize(txt)
                local fc = corFonte or V4(1,1,1,1)
                DL:AddText(V2(ax + 5 + (offX or 0), ay + (altura - ts.y)/2 + (offY or 0)), u32(fc), txt)
                imgui.SetWindowFontScale(1.0)
                imgui.PopFont()
            end
        end
        barS2(bXB - lS2 - r2, bYB, lS2, hS2, hp, 100, clr.hp[0], clr.hp[1], clr.hp[2], icns["heart"], V4(clr.ic[0],clr.ic[1],clr.ic[2],clr.ic[3]), V4(corFonteHP[0],corFonteHP[1],corFonteHP[2],corFonteHP[3]), hfx[0], hfy[0], hfs[0])
        if ap > 0 or hpe[0] then
            barS2(bXB - lS2 - r2, bYB + spYS2, lS2, hS2, ap, 100, clr.ap[0], clr.ap[1], clr.ap[2], icns["shield"], V4(clr.ic2[0],clr.ic2[1],clr.ic2[2],clr.ic2[3]), V4(corFonteAP[0],corFonteAP[1],corFonteAP[2],corFonteAP[3]), afx[0], afy[0], afs[0])
        end
        -- Barra de estamina removida no Estilo 2
    end
    imgui.PushFont(fDin)
    local sc = fdin[0] / 15.0
    imgui.SetWindowFontScale(sc)
    local money = getPlayerMoney(PLAYER_HANDLE)
    local mTxt = "$" .. intSep(money)
    local mw = imgui.CalcTextSize(mTxt).x
    local yMoneyBase = st2[0] and (bYB_base + (hS2 or 25*sM)*3 + 20*sM) or (bYB_base + 70 * sM + 30)
    local yMoney = yMoneyBase + dy[0]
    DL:AddText(V2(x - r - 30 - mw + dx[0], yMoney),
        u32(V4(clr.d[0],clr.d[1],clr.d[2],clr.d[3])), mTxt)
    imgui.SetWindowFontScale(1.0)
    imgui.PopFont()
    if ma[0] then
        local cur = getCurrentCharWeapon(PLAYER_PED)
        if cur ~= lastWep then
            clipAmmo = getClip()
            totalAmmo = getAmmoInCharWeapon(PLAYER_PED, cur)
            lastWep = cur
            isReload = false
            reloadTimer = 0
        end
        if isCharShooting(PLAYER_PED) and not isReload and clipAmmo > 0 then
            clipAmmo = clipAmmo - 1
        end
        if clipAmmo == 0 and not isReload then
            isReload = true
            reloadTimer = os.clock()
        end
        if isReload and os.clock() - reloadTimer >= 0.9 then
            totalAmmo = getAmmoInCharWeapon(PLAYER_PED, cur)
            clipAmmo = getClip()
            isReload = false
        end
        if totalAmmo > 0 then
            imgui.PushFont(fMun)
            local sc2 = fma[0] / 15.0
            imgui.SetWindowFontScale(sc2)
            local aTxt = string.format("%d/%d", clipAmmo, totalAmmo)
            local ts = imgui.CalcTextSize(aTxt)
            local yAmmo = st2[0] and (yMoneyBase + 30*sM + my[0]) or (bYB_base + r + 20 + my[0])
            DL:AddText(V2(x - ts.x/2 + mx[0], yAmmo),
                u32(V4(clr.m[0],clr.m[1],clr.m[2],clr.m[3])), aTxt)
            imgui.SetWindowFontScale(1.0)
            imgui.PopFont()
        end
    end
    local ss = te[0]
    if me[0] and starTex then
        if st2[0] then
            local ey = yMoneyBase + 60*sM + e2y[0]
            for i=1,6 do
                local ex = x + ox[0] - (i-1) * (ss*2 + ee[0]) - ss*3 + e2x[0]
                DL:AddImage(starTex, V2(ex - ss, ey - ss), V2(ex + ss, ey + ss), nil, nil,
                    u32(V4(clr.e[0],clr.e[1],clr.e[2],clr.e[3])))
            end
        else
            if mel[0] then
                for i=1,6 do
                    local ex = x + ox[0] - (i-1) * (ss*2 + ee[0])
                    local ey = y + oy[0] - 50
                    DL:AddImage(starTex, V2(ex - ss, ey - ss), V2(ex + ss, ey + ss), nil, nil,
                        u32(V4(clr.e[0],clr.e[1],clr.e[2],clr.e[3])))
                end
            else
                local angs = {40,60,80,100,120,140}
                local ext = 30
                for i=1,6 do
                    local ang = angs[i] - 180
                    local ex = x + ox[0] - (r + ext + ee[0]) * math.sin(math.rad(ang))
                    local ey = y + oy[0] - (r + ext) * math.cos(math.rad(ang))
                    DL:AddImage(starTex, V2(ex - ss, ey - ss), V2(ex + ss, ey + ss), nil, nil,
                        u32(V4(clr.e[0],clr.e[1],clr.e[2],clr.e[3])))
                end
            end
        end
    end
end
local function drawMira(DL, rx, ry)
    if not miraAtivada[0] then return end
    local cx, cy = rx/2 + miraX[0], ry/2 + miraY[0]
    local sz = miraTamanho[0]
    local w = miraLargura[0]
    local t = miraTipo[0]
    local c = u32(V4(corMira[0], corMira[1], corMira[2], corMira[3]))
    local cb = u32(V4(corBordaMira[0], corBordaMira[1], corBordaMira[2], corBordaMira[3]))
    local b = miraBordaAtivada[0] and miraTamanhoBorda[0] or 0
    local function L(x1,y1,x2,y2,clr,th) DL:AddLine(V2(x1,y1), V2(x2,y2), clr, th) end
    local function C(x,y,r,clr,th) DL:AddCircle(V2(x,y), r, clr, 0, th) end
    local function CF(x,y,r,clr) DL:AddCircleFilled(V2(x,y), r, clr) end
    local function R(x1,y1,x2,y2,clr,th) DL:AddRect(V2(x1,y1), V2(x2,y2), clr, 0, 0, th) end
    
    local function drawShape(off, clr, th)
        local s = sz + off
        local m = s / 2
        local ww = th
        if t == 0 then -- Cruz Simples
            L(cx-s, cy, cx+s, cy, clr, ww); L(cx, cy-s, cx, cy+s, clr, ww)
        elseif t == 1 then -- Cruz Dupla
            L(cx-s, cy, cx-m+off, cy, clr, ww); L(cx+m-off, cy, cx+s, cy, clr, ww)
            L(cx, cy-s, cx, cy-m+off, clr, ww); L(cx, cy+m-off, cx, cy+s, clr, ww)
        elseif t == 2 then -- Circulo
            C(cx, cy, s, clr, ww)
        elseif t == 3 then -- Quadrado
            R(cx-s, cy-s, cx+s, cy+s, clr, ww)
        elseif t == 4 then -- X
            L(cx-s, cy-s, cx+s, cy+s, clr, ww); L(cx+s, cy-s, cx-s, cy+s, clr, ww)
        elseif t == 5 then -- Ponto
            CF(cx, cy, s/3, clr)
        elseif t == 6 then -- Alvo
            C(cx, cy, s, clr, ww); C(cx, cy, s/2, clr, ww)
            L(cx-s, cy, cx+s, cy, clr, ww); L(cx, cy-s, cx, cy+s, clr, ww)
        elseif t == 7 then -- Diamante
            local p = {V2(cx, cy-s), V2(cx+s, cy), V2(cx, cy+s), V2(cx-s, cy)}
            for i=1,4 do DL:AddLine(p[i], p[i%4+1], clr, ww) end
        elseif t == 8 then -- Estrela
            local p = {}
            for i=1,5 do
                local a = math.rad(i*144-90)
                table.insert(p, V2(cx+math.cos(a)*s, cy+math.sin(a)*s))
            end
            for i=1,5 do DL:AddLine(p[i], p[i%5+1], clr, ww) end
        elseif t == 9 then -- Triangulo
            local p = {V2(cx, cy-s), V2(cx+s, cy+s), V2(cx-s, cy+s)}
            for i=1,3 do DL:AddLine(p[i], p[i%3+1], clr, ww) end
        elseif t == 10 then -- Cruz Fina
            L(cx-s, cy, cx+s, cy, clr, 1+off*2); L(cx, cy-s, cx, cy+s, clr, 1+off*2)
        elseif t == 11 then -- Cruz Grossa
            L(cx-s, cy, cx+s, cy, clr, 5+off*2); L(cx, cy-s, cx, cy+s, clr, 5+off*2)
        elseif t == 12 then -- Circulo Duplo
            C(cx, cy, s, clr, ww); C(cx, cy, s/2, clr, ww)
        elseif t == 13 then -- Quadrado Vazado
            R(cx-s, cy-s, cx+s, cy+s, clr, ww)
        elseif t == 14 then -- X com Ponto
            L(cx-s, cy-s, cx+s, cy+s, clr, ww); L(cx+s, cy-s, cx-s, cy+s, clr, ww); CF(cx, cy, s/4, clr)
        elseif t == 15 then -- Alvo Precisão
            C(cx, cy, s, clr, ww); L(cx-s, cy, cx-m, cy, clr, ww); L(cx+m, cy, cx+s, cy, clr, ww)
            L(cx, cy-s, cx, cy-m, clr, ww); L(cx, cy+m, cx, cy+s, clr, ww)
        elseif t == 16 then -- Diamante Vazado
            local p = {V2(cx, cy-s), V2(cx+s, cy), V2(cx, cy+s), V2(cx-s, cy)}
            for i=1,4 do DL:AddLine(p[i], p[i%4+1], clr, ww) end
        elseif t == 17 then -- Estrela 6 Pontas
            local p = {}
            for i=1,6 do
                local a = math.rad(i*60-90)
                table.insert(p, V2(cx+math.cos(a)*s, cy+math.sin(a)*s))
            end
            for i=1,6 do DL:AddLine(p[i], p[i%6+1], clr, ww) end
        elseif t == 18 then -- Triangulo Invertido
            local p = {V2(cx, cy+s), V2(cx+s, cy-s), V2(cx-s, cy-s)}
            for i=1,3 do DL:AddLine(p[i], p[i%3+1], clr, ww) end
        elseif t == 19 then -- Cruz Suíça
            L(cx-s, cy, cx+s, cy, clr, ww); L(cx, cy-s, cx, cy+s, clr, ww)
            L(cx-m, cy-m, cx+m, cy+m, clr, ww); L(cx+m, cy-m, cx-m, cy+m, clr, ww)
        elseif t == 20 then -- Alvo Tático
            C(cx, cy, s, clr, ww); C(cx, cy, s/4, clr, ww)
            L(cx-s, cy, cx-m, cy, clr, ww); L(cx+m, cy, cx+s, cy, clr, ww)
            L(cx, cy-s, cx, cy-m, clr, ww); L(cx, cy+m, cx, cy+s, clr, ww)
        elseif t == 21 then -- Ponto Duplo
            CF(cx, cy, s/4, clr); C(cx, cy, s/2, clr, ww)
        elseif t == 22 then -- Círculo com Cruz
            C(cx, cy, s, clr, ww); L(cx-s, cy, cx+s, cy, clr, ww); L(cx, cy-s, cx, cy+s, clr, ww)
        elseif t == 23 then -- Quadrado com Cruz
            R(cx-s, cy-s, cx+s, cy+s, clr, ww)
            L(cx-s, cy, cx+s, cy, clr, ww); L(cx, cy-s, cx, cy+s, clr, ww)
        elseif t == 24 then -- X com Círculo
            L(cx-s, cy-s, cx+s, cy+s, clr, ww); L(cx+s, cy-s, cx-s, cy+s, clr, ww); C(cx, cy, s, clr, ww)
        elseif t == 25 then -- Hexágono
            local p = {}
            for i=1,6 do
                local a = math.rad(i*60-90)
                table.insert(p, V2(cx+math.cos(a)*s, cy+math.sin(a)*s))
            end
            for i=1,6 do DL:AddLine(p[i], p[i%6+1], clr, ww) end
        elseif t == 26 then -- Octógono
            local p = {}
            for i=1,8 do
                local a = math.rad(i*45-90)
                table.insert(p, V2(cx+math.cos(a)*s, cy+math.sin(a)*s))
            end
            for i=1,8 do DL:AddLine(p[i], p[i%8+1], clr, ww) end
        elseif t == 27 then -- Círculo Segmentado
            C(cx, cy, s, clr, ww)
            for i=0,7 do
                local a = math.rad(i*45)
                L(cx+math.cos(a)*s, cy+math.sin(a)*s, cx+math.cos(a)*m, cy+math.sin(a)*m, clr, ww)
            end
        elseif t == 28 then -- Cruz Tática
            L(cx-s, cy, cx-m, cy, clr, ww); L(cx+m, cy, cx+s, cy, clr, ww)
            L(cx, cy-s, cx, cy-m, clr, ww); L(cx, cy+m, cx, cy+s, clr, ww)
            L(cx-m, cy-m, cx+m, cy+m, clr, ww); L(cx+m, cy-m, cx-m, cy+m, clr, ww)
        elseif t == 29 then -- Alvo Avançado
            C(cx, cy, s, clr, ww); C(cx, cy, m, clr, ww); C(cx, cy, s/4, clr, ww)
            L(cx-s, cy, cx+s, cy, clr, ww); L(cx, cy-s, cx, cy+s, clr, ww)
        end
    end

    if b > 0 then drawShape(b, cb, w+b*2) end
    drawShape(0, c, w)
end
local skinIdInput = imgui.new.int(skinId)
imgui.OnFrame(function() return not isPauseMenuActive() end, function()
    if hudRGB[0] then
        updateRainbowColors()
        applyRGBColors()
    else
        restoreSavedColors()
    end
    if weatherEnabled[0] then
        setTimeOfDay(weatherTime[0], 0)
        forceWeatherNow(weatherId[0])
    end
    if skinEnabled and skinId ~= 0 then
        local currentSkin = getCharModel(PLAYER_PED)
        if currentSkin ~= skinId then
            setPlayerSkin(skinId)
        end
    end
    updateJumpCircles()
    if oh[0] then
        displayHud(true)
        return
    end
    -- Se estiver no Estilo 2, precisamos do HUD original para o Fist
    if st2[0] then
        displayHud(true)
    else
        displayHud(false)
    end
    staminaped()
    if savePend then
        saveTimer = saveTimer + 1
        if saveTimer > 10 then saveCfg() end
    end
    local rx,ry = getScreenResolution()
    imgui.SetNextWindowPos(V2(0,0))
    imgui.SetNextWindowSize(V2(rx,ry))
    imgui.SetNextWindowBgAlpha(0)
    imgui.PushStyleColor(imgui.Col.WindowBg, V4(0,0,0,0))
    imgui.PushStyleColor(imgui.Col.Border, V4(0,0,0,0))
    imgui.PushStyleColor(imgui.Col.BorderShadow, V4(0,0,0,0))
    imgui.Begin("##hud", nil,
        imgui.WindowFlags.NoTitleBar + imgui.WindowFlags.NoResize +
        imgui.WindowFlags.NoScrollbar + imgui.WindowFlags.NoInputs +
        imgui.WindowFlags.NoMove + imgui.WindowFlags.NoCollapse)
    imgui.PopStyleColor(3)
    imgui.PushFont(fHud)
    local DL = imgui.GetBackgroundDrawList()
    local io = imgui.GetIO()
    local mx,my = io.MousePos.x, io.MousePos.y
    if sets[0] or exibir_menu[0] then io.MouseDrawCursor = true else io.MouseDrawCursor = false end
    
    if showSplash then
        if os.clock() - splashTimer < 3.0 then
            imgui.PushFont(fHud)
            local ts = imgui.CalcTextSize(splashText)
            local sX, sY = getScreenResolution()
            local pX, pY = (sX - ts.x * 3) / 2, (sY - ts.y * 3) / 2
            DL:AddText(V2(pX + 2, pY + 2), 0xFF000000, splashText)
            imgui.SetWindowFontScale(3.0)
            DL:AddText(V2(pX, pY), 0xFFFFFFFF, splashText)
            imgui.SetWindowFontScale(1.0)
            imgui.PopFont()
        else
            showSplash = false
        end
    end

    if mMode then
        if io.MouseDown[0] then
            if not mDrag then
                mDrag = true; mMX0 = mx; mMY0 = my
                mPX0 = posX[0]; mPY0 = posY[0]
            end
            posX[0] = math.floor(mPX0 + (mx - mMX0))
            posY[0] = math.floor(mPY0 + (my - mMY0))
            qSave()
        else
            if mDrag then
                mDrag = false
                mMode = false
                qSave()
            end
        end
        local px,py = posX[0], posY[0]
        DL:AddCircleFilled(V2(px,py),22,u32(V4(0.3,0.3,0.3,0.15)))
        DL:AddCircle(V2(px,py),22,u32(ACENT),32,2)
        DL:AddCircle(V2(px,py),5,u32(ACENT),16,3)
        DL:AddLine(V2(px-28,py), V2(px-14,py), u32(ACENT), 1.5)
        DL:AddLine(V2(px+14,py), V2(px+28,py), u32(ACENT), 1.5)
        DL:AddLine(V2(px,py-28), V2(px,py-14), u32(ACENT), 1.5)
        DL:AddLine(V2(px,py+14), V2(px,py+28), u32(ACENT), 1.5)
        local dica = mDrag and T("sf") or T("ma")
        local ts = imgui.CalcTextSize(dica)
        DL:AddRectFilled(V2(rx/2 - ts.x/2 - 12, ry - 68), V2(rx/2 + ts.x/2 + 12, ry - 42), u32(V4(0,0,0,0.75)), 8)
        DL:AddText(V2(rx/2 - ts.x/2, ry - 65), u32(ACENT), dica)
    end
    local x,y = posX[0],posY[0]
    local sM = ({1.0,1.2,1.4})[hsz[0] + 1] or 1.2
    local r = st2[0] and 0 or (70 * sM)
    local pol = r * 1.5
    local gIdx = math.floor(os.clock() / 0.07) % (totalF > 0 and totalF or 1)
    local wid = getCurrentCharWeapon(PLAYER_PED)
    if not st2[0] then
        sprCircle(DL,x,y,r,pol)
        wep1(DL,x,y,r,sM,gIdx,wid)
    else
        wep1(DL,x,y,r,sM,gIdx,wid)
        wep2(DL,x,y,gIdx,wid)
    end
    bars(DL,x,y,r,sM)
    drawMira(DL, rx, ry)
    local rainbowColors = {V4(1,0,0,1), V4(1,0.5,0,1), V4(1,1,0,1), V4(0,1,0,1), V4(0,0,1,1), V4(0.3,0,0.5,1)}
    rainbowAlpha = rainbowAlpha + (0.02 * rainbowFade)
    if rainbowAlpha >= 1.0 then rainbowFade = -1 elseif rainbowAlpha <= 0.3 then rainbowFade = 1; rainbowIdx = (rainbowIdx % 6) + 1 end
    local rc = rainbowColors[rainbowIdx]
    -- Texto RGB removido
    imgui.PopFont()

    if showUpdateScreen then
        local rx, ry = getScreenResolution()
        DL:AddRectFilled(V2(0, 0), V2(rx, ry), 0xFF000000) -- Tela preta total
        
        local btnW, btnH = 400 * MDS, 100 * MDS
        local btnX, btnY = (rx - btnW) / 2, (ry - btnH) / 2
        
        -- Botão Vermelho
        DL:AddRectFilled(V2(btnX, btnY), V2(btnX + btnW, btnY + btnH), 0xFF0000FF, 10) -- Vermelho (ABGR: FF0000FF)
        
        local txt = "ATUALIZAR"
        imgui.PushFont(fHud)
        imgui.SetWindowFontScale(2.0)
        local ts = imgui.CalcTextSize(txt)
        DL:AddText(V2(btnX + (btnW - ts.x*2)/2, btnY + (btnH - ts.y*2)/2), 0xFFFFFFFF, txt)
        imgui.SetWindowFontScale(1.0)
        imgui.PopFont()
        
        -- Lógica de clique no botão
        if io.MouseClicked[0] then
            if io.MousePos.x >= btnX and io.MousePos.x <= btnX + btnW and
               io.MousePos.y >= btnY and io.MousePos.y <= btnY + btnH then
                -- Abrir link no Android via FFI
                local link = "https://www.mediafire.com/file/uwiuygapciqb38b/DEPENDENCIAS+V6+by+Shellder.zip/file"
                ffi.gc(ffi.cast("const char*", link), nil)
                gta._Z12AND_OpenLinkPKc(link)
            end
        end
        
        -- Bloquear o restante do HUD se a tela de atualização estiver ativa
        imgui.End()
        return
    end

    imgui.End()
end)
local function Sep()
    imgui.Dummy(V2(0, 5 * MDS))
end
local function Tgl(est, uid)
    local dl = imgui.GetWindowDrawList()
    local p = imgui.GetCursorScreenPos()
    local tw = 38 * MDS
    local th = 20 * MDS
    local r = th * 0.5
    local trackCol = est and u32(V4(0.45,0.45,0.45,1)) or u32(V4(0.18,0.18,0.18,1))
    dl:AddRectFilled(V2(p.x, p.y), V2(p.x + tw, p.y + th), trackCol, r)
    dl:AddRect(V2(p.x, p.y), V2(p.x + tw, p.y + th), u32(V4(0.30,0.30,0.30,1)), r, 15, 1)
    local mg = 2.5 * MDS
    local knobX = est and (p.x + tw - r) or (p.x + r)
    local knobCol = est and u32(V4(0.90,0.90,0.90,1)) or u32(V4(0.50,0.50,0.50,1))
    dl:AddCircleFilled(V2(knobX, p.y + r), r - mg, knobCol, 20)
    imgui.InvisibleButton("##tg"..uid, V2(tw, th))
    if imgui.IsItemClicked() then return not est end
    return est
end
local function TL(l, est, uid)
    local v = Tgl(est, uid)
    imgui.SameLine(0, 8 * MDS)
    imgui.TextColored(V4(0.80,0.80,0.80,1), l)
    return v
end
local function SI(l, v, mn, mx)
    imgui.TextColored(TXT_M, l)
    imgui.PushItemWidth(imgui.GetContentRegionAvail().x)
    local ch = imgui.SliderInt("##si"..l, v, mn, mx)
    imgui.PopItemWidth()
    return ch
end
local function SF(l, v, mn, mx, fmt)
    imgui.TextColored(TXT_M, l)
    imgui.PushItemWidth(imgui.GetContentRegionAvail().x)
    local ch = imgui.SliderFloat("##sf"..l, v, mn, mx, fmt or "%.1f")
    imgui.PopItemWidth()
    return ch
end
function DrawInterfaceTeam(wdl)
    imgui.TextColored(V4(1, 1, 1, 1), "INTERFACE TEAM (")
    imgui.SameLine(0, 0)
    imgui.TextColored(V4(1, 0, 0, 1), st2[0] and "2" or "1")
    imgui.SameLine(0, 0)
    imgui.TextColored(V4(1, 1, 1, 1), ")")
    Sep()
    imgui.Spacing()
    local pst = st2[0]
    local v3 = TL("Trocar Estilo", st2[0], "st")
    if v3 ~= pst then
        local oldX, oldY = posX[0], posY[0]
        saveStyle(pst)
        st2[0] = v3
        loadStyle(st2[0])
        posX[0], posY[0] = oldX, oldY
        qSave()
    end
    imgui.Dummy(V2(0, 10 * MDS))
end

function DrawGeralS1(wdl)
    imgui.TextColored(ACENT, T("g"))
    Sep()
    imgui.Spacing()
    local v1 = TL(T("oh"), oh[0], "oh")
    if v1 ~= oh[0] then oh[0] = v1; qSave() end
    imgui.Spacing()
    local v2 = TL(T("vals"), vals[0], "va")
    if v2 ~= vals[0] then vals[0] = v2; qSave() end
    imgui.Spacing()
    if not st2[0] then
        local v5 = TL(T("cspr"), cspr[0], "cs")
        if v5 ~= cspr[0] then cspr[0] = v5; qSave() end
        imgui.Spacing()
    end
    imgui.TextColored(TXT_M, T("th"))
    local szL = {T("pq"), T("md"), T("gr")}
    for i = 1,3 do
        if i > 1 then imgui.SameLine(0, 6 * MDS) end
        local sel = hsz[0] == i - 1
        local p = imgui.GetCursorScreenPos()
        local bw = 58 * MDS
        local bh = 22 * MDS
        local bgCol = sel and u32(V4(0.35,0.35,0.35,0.35)) or u32(V4(0.16,0.16,0.16,1))
        wdl:AddRectFilled(p, V2(p.x+bw, p.y+bh), bgCol, 8)
        if sel then
            wdl:AddRect(p, V2(p.x+bw, p.y+bh), u32(V4(0.55,0.55,0.55,0.7)), 8, 0, 1.2)
        end
        local ts = imgui.CalcTextSize(szL[i])
        wdl:AddText(V2(p.x+(bw-ts.x)*0.5, p.y+(bh-ts.y)*0.5),
            sel and u32(V4(0.92,0.92,0.92,1)) or u32(V4(0.55,0.55,0.55,1)), szL[i])
        imgui.InvisibleButton("##sz"..i, V2(bw, bh))
        if imgui.IsItemClicked() then hsz[0] = i-1; qSave() end
    end
    imgui.Spacing()
end
function DrawGeralS2(wdl)
    imgui.TextColored(TXT_M, T("mp"))
    local bw = (imgui.GetContentRegionAvail().x - 4 * MDS) / 2
    local bh = 26 * MDS
    local at = mMode
    local pBtn = imgui.GetCursorScreenPos()
    local bgA = at and u32(V4(0.35,0.35,0.35,0.5)) or u32(V4(0.16,0.16,0.16,1))
    wdl:AddRectFilled(pBtn, V2(pBtn.x+bw, pBtn.y+bh), bgA, 8)
    wdl:AddRect(pBtn, V2(pBtn.x+bw, pBtn.y+bh), u32(V4(at and 0.65 or 0.28, at and 0.65 or 0.28, at and 0.65 or 0.28, at and 1 or 0.5)), 8, 0, 1.2)
    local lbl = at and T("aa") or T("arr")
    local tsB = imgui.CalcTextSize(lbl)
    wdl:AddText(V2(pBtn.x+(bw-tsB.x)*0.5, pBtn.y+(bh-tsB.y)*0.5),
        at and u32(V4(0.92,0.92,0.92,1)) or u32(V4(0.60,0.60,0.60,1)), lbl)
    imgui.InvisibleButton("##mv", V2(bw, bh))
    if imgui.IsItemClicked() then
        mSlider = false
        if not mMode then
            mMode = true; mDrag = false
            mPX0 = posX[0]; mPY0 = posY[0]
        else
            mMode = false
        end
        sets[0] = false
    end
    imgui.SameLine(0, 4 * MDS)
    local as = mSlider
    local pBtn2 = imgui.GetCursorScreenPos()
    local bgB = as and u32(V4(0.35,0.35,0.35,0.5)) or u32(V4(0.16,0.16,0.16,1))
    wdl:AddRectFilled(pBtn2, V2(pBtn2.x+bw, pBtn2.y+bh), bgB, 8)
    wdl:AddRect(pBtn2, V2(pBtn2.x+bw, pBtn2.y+bh), u32(V4(as and 0.65 or 0.28, as and 0.65 or 0.28, as and 0.65 or 0.28, as and 1 or 0.5)), 8, 0, 1.2)
    local lbl2 = as and T("sa") or T("sxy")
    local tsB2 = imgui.CalcTextSize(lbl2)
    wdl:AddText(V2(pBtn2.x+(bw-tsB2.x)*0.5, pBtn2.y+(bh-tsB2.y)*0.5),
        as and u32(V4(0.92,0.92,0.92,1)) or u32(V4(0.60,0.60,0.60,1)), lbl2)
    imgui.InvisibleButton("##sl", V2(bw, bh))
    if imgui.IsItemClicked() then mMode = false; mSlider = not mSlider end
    imgui.Dummy(V2(imgui.GetContentRegionAvail().x, 2 * MDS))
    if mSlider then
        if SI(T("px"), posX, 100, 3000) then qSave() end
        if SI(T("py"), posY, 100, 2000) then qSave() end
    end
    imgui.Spacing()
end
function DrawGeralS3()
    imgui.TextColored(ACENT, T("bs"))
    Sep()
    local mj = TL("Mover Barras Juntas", moverJuntos[0], "mj")
    if mj ~= moverJuntos[0] then moverJuntos[0] = mj; qSave() end
    imgui.Spacing()
    
    if SI(T("lb"), lb, 50, 500) then qSave() end
    if SI(T("ab"), ab, 2, 100) then qSave() end
    
    if moverJuntos[0] then
        if SI(T("bx"), bx, -2000, 2000) then qSave() end
        if SI(T("by"), by, -2000, 2000) then qSave() end
    else
        imgui.TextColored(TXT_M, "Controles individuais:")
        if SI("HP X", bx, -2000, 2000) then qSave() end
        if SI("HP Y", by, -2000, 2000) then qSave() end
        -- Nota: Para controle 100% separado de cada barra, 
        -- precisaríamos de variáveis extras no cfg.st1/st2.
    end
    
    if SI(T("fv"), fv, 8, 40) then qSave() end
    imgui.Spacing()
    -- Posição das fontes de vida e colete
    imgui.TextColored(ACENT, "Fontes Vida/Colete")
    Sep()
    local mfj = TL("Mover Fontes Juntas", moverFontesJuntas[0], "mfj")
    if mfj ~= moverFontesJuntas[0] then moverFontesJuntas[0] = mfj; qSave() end
    imgui.Spacing()
    if moverFontesJuntas[0] then
        if SI("Fonte X", hfx, -500, 500) then
            afx[0] = hfx[0]
            qSave()
        end
        if SI("Fonte Y", hfy, -500, 500) then
            afy[0] = hfy[0]
            qSave()
        end
    else
        if SI("Fonte Vida X", hfx, -500, 500) then qSave() end
        if SI("Fonte Vida Y", hfy, -500, 500) then qSave() end
        if SI("Fonte Colete X", afx, -500, 500) then qSave() end
        if SI("Fonte Colete Y", afy, -500, 500) then qSave() end
    end
    imgui.Spacing()
    imgui.TextColored(TXT_M, "Tamanho Fonte Vida")
    imgui.PushItemWidth(imgui.GetContentRegionAvail().x)
    if imgui.SliderInt("##fszHP", hfs, 6, 60) then qSave() end
    imgui.PopItemWidth()
    imgui.TextColored(TXT_M, "Tamanho Fonte Colete")
    imgui.PushItemWidth(imgui.GetContentRegionAvail().x)
    if imgui.SliderInt("##fszAP", afs, 6, 60) then qSave() end
    imgui.PopItemWidth()
    imgui.Spacing()
    if st2[0] then
        imgui.TextColored(ACENT, "Redondeza das Bordas")
        Sep()
        local bw2 = (imgui.GetContentRegionAvail().x - 4 * MDS) / 2
        local bh2 = 26 * MDS
        if imgui.Button("-##rnd", V2(bw2, bh2)) then
            if barS2Rnd[0] > 0 then barS2Rnd[0] = barS2Rnd[0] - 1; qSave() end
        end
        imgui.SameLine(0, 4 * MDS)
        if imgui.Button("+##rnd", V2(bw2, bh2)) then
            if barS2Rnd[0] < 30 then barS2Rnd[0] = barS2Rnd[0] + 1; qSave() end
        end
        imgui.TextColored(TXT_M, string.format("Valor: %d", barS2Rnd[0]))
        imgui.Spacing()
    end
    if not st2[0] then
        local vrd = TL(T("rd"), rd[0], "rd")
        if vrd ~= rd[0] then rd[0] = vrd; qSave() end
        if rd[0] then
            imgui.Spacing()
            if SI(T("rdlv"), rdlv, 0, 30) then qSave() end
        end
        imgui.Spacing()
    end
    local vhp = TL(T("hpe"), hpe[0], "hp")
    if vhp ~= hpe[0] then hpe[0] = vhp; qSave() end
    imgui.Spacing()
    if not st2[0] then
        local vfl = TL(T("mfl"), mfl[0], "fl")
        if vfl ~= mfl[0] then mfl[0] = vfl; qSave() end
        imgui.Spacing()
    end
    local vic = TL(T("iv").."/"..T("ia"), ic[0], "ic")
    if vic ~= ic[0] then ic[0] = vic; qSave() end
    imgui.Spacing()
end
function DrawGeralS4()
    imgui.TextColored(ACENT, T("ds"))
    Sep()
    if SI(T("dx"), dx, -500, 500) then qSave() end
    if SI(T("dy"), dy, -500, 500) then qSave() end
    if SI(T("fdin"), fdin, 8, 60) then qSave() end
    imgui.Spacing()
    imgui.TextColored(ACENT, T("m"))
    Sep()
    local vm = TL(T("ma"), ma[0], "ma")
    if vm ~= ma[0] then ma[0] = vm; qSave() end
    if ma[0] then
        imgui.Spacing()
        if SI(T("mx"), mx, -500, 500) then qSave() end
        if SI(T("my"), my, -500, 500) then qSave() end
        if SI(T("fma"), fma, 8, 50) then qSave() end
    end
    imgui.Spacing()
end
function DrawGeralS5(wdl)
    imgui.TextColored(ACENT, T("armas"))
    Sep()
    local vpa = TL(T("pa"), pa[0], "pa")
    if vpa ~= pa[0] then
        pa[0] = vpa
        qSave()
    end
    if pa[0] then
        imgui.Spacing()
        if not st2[0] then
            if SF(T("tpa"), psz, 1.0, 2.0, "%.1f") then qSave() end
            imgui.Spacing()
        end
        imgui.TextColored(TXT_M, T("fg"))
        Sep()
        if gifCount > 0 then
            local bwGif = (imgui.GetContentRegionAvail().x - (gifCount-1)*4*MDS) / math.max(gifCount,1)
            bwGif = math.min(bwGif, 60*MDS)
            local bh = 22*MDS
            for i = 1, gifCount do
                if i > 1 then imgui.SameLine(0, 4*MDS) end
                local sel = fistIdx[0] == i
                local p = imgui.GetCursorScreenPos()
                local label = "fist"..i
                wdl:AddRectFilled(p, V2(p.x+bwGif,p.y+bh), sel and u32(V4(0.35,0.35,0.35,0.4)) or u32(V4(0.16,0.16,0.16,1)), 7)
                if sel then wdl:AddRect(p, V2(p.x+bwGif,p.y+bh), u32(V4(0.55,0.55,0.55,0.7)), 7, 0, 1.2) end
                local ts = imgui.CalcTextSize(label)
                wdl:AddText(V2(p.x+(bwGif-ts.x)*0.5, p.y+(bh-ts.y)*0.5),
                    sel and u32(V4(0.92,0.92,0.92,1)) or u32(V4(0.55,0.55,0.55,1)), label)
                imgui.InvisibleButton("##gif"..i, V2(bwGif, bh))
                if imgui.IsItemClicked() then
                    fistIdx[0] = i
                    carregarFramesGif(i)
                    cfg.main.fistIdx = i
                    qSave()
                end
            end
            imgui.Spacing()
        else
            imgui.TextColored(TXT_M, T("fn"))
            imgui.Spacing()
        end
        if not st2[0] then
            imgui.TextColored(TXT_M, T("ft"))
            local isCirc = fistCircular[0]
            local bwT = (imgui.GetContentRegionAvail().x - 4*MDS) / 2
            local bhT = 22*MDS
            local tipos = {T("fq"), T("fc")}
            for i,lbl in ipairs(tipos) do
                if i > 1 then imgui.SameLine(0, 4*MDS) end
                local sel = (i == 2) == isCirc
                local p = imgui.GetCursorScreenPos()
                wdl:AddRectFilled(p, V2(p.x+bwT,p.y+bhT), sel and u32(V4(0.35,0.35,0.35,0.4)) or u32(V4(0.16,0.16,0.16,1)), 7)
                if sel then wdl:AddRect(p, V2(p.x+bwT,p.y+bhT), u32(V4(0.55,0.55,0.55,0.7)), 7, 0, 1.2) end
                local ts = imgui.CalcTextSize(lbl)
                wdl:AddText(V2(p.x+(bwT-ts.x)*0.5, p.y+(bhT-ts.y)*0.5),
                    sel and u32(V4(0.92,0.92,0.92,1)) or u32(V4(0.55,0.55,0.55,1)), lbl)
                imgui.InvisibleButton("##ftype"..i, V2(bwT, bhT))
                if imgui.IsItemClicked() then
                    fistCircular[0] = (i == 2)
                    cfg.main.fistCircular = fistCircular[0]
                    qSave()
                end
            end
            imgui.Spacing()
            imgui.TextColored(TXT_M, T("fb"))
            imgui.PushItemWidth(imgui.GetContentRegionAvail().x)
            if imgui.SliderInt("##fbs", fistBordeScale, 80, 200) then
                cfg.main.fistBordeScale = fistBordeScale[0]
                qSave()
            end
            imgui.PopItemWidth()
            imgui.Spacing()
        end
    end
    imgui.Spacing()
    if not st2[0] then
        if SI(T("ix"), ix, -300, 300) then qSave() end
        if SI(T("iy"), iy, -300, 300) then qSave() end
    end
    imgui.Spacing()
    imgui.TextColored(ACENT, T("e"))
    Sep()
    local ves = TL(T("me"), me[0], "me")
    if ves ~= me[0] then me[0] = ves; qSave() end
    if me[0] then
        imgui.Spacing()
        if not st2[0] then
            local vel = TL(T("mel"), mel[0], "ml")
            if vel ~= mel[0] then mel[0] = vel; qSave() end
            imgui.Spacing()
        end
        if SI(T("te"), te, 4, 30) then qSave() end
        if not st2[0] then
            if SI(T("ox"), ox, -500, 500) then qSave() end
            if SI(T("oy"), oy, -500, 500) then qSave() end
        end
        if SI(T("ee"), ee, -20, 50) then qSave() end
        if st2[0] then
            if SI(T("e2x"), e2x, -500, 500) then qSave() end
            if SI(T("e2y"), e2y, -500, 500) then qSave() end
        end
    end
    imgui.Spacing()
end
function DrawExtraT(wdl)
    imgui.TextColored(ACENT, T("cl"))
    Sep()
    local vcl = TL(T("cl_at"), weatherEnabled[0], "cl_at")
    if vcl ~= weatherEnabled[0] then weatherEnabled[0] = vcl; qSave() end
    if weatherEnabled[0] then
        imgui.Spacing()
        imgui.TextColored(TXT_M, T("cl_time"))
        imgui.PushItemWidth(200 * MDS)
        if imgui.SliderInt("##time", weatherTime, 0, 23) then qSave() end
        imgui.Spacing()
        imgui.TextColored(TXT_M, T("cl_id"))
        if imgui.SliderInt("##weather", weatherId, 0, 45) then qSave() end
        imgui.PopItemWidth()
    end
    imgui.Spacing()
    imgui.TextColored(ACENT, T("skin"))
    Sep()
    local vskin = TL(T("skin_at"), skinEnabled, "skin_at")
    if vskin ~= skinEnabled then skinEnabled = vskin; qSave() end
    if skinEnabled then
        imgui.Spacing()
        imgui.TextColored(TXT_M, T("skin_id"))
        imgui.PushItemWidth(150 * MDS)
        if imgui.InputInt("##skin_id", skinIdInput) then
            skinId = skinIdInput[0]
            qSave()
        end
        imgui.PopItemWidth()
        imgui.SameLine(0, 5 * MDS)
        if imgui.Button(T("skin_apply"), V2(60 * MDS, 0)) then
            skinId = skinIdInput[0]
            if skinId ~= 0 then
                setPlayerSkin(skinId)
            end
            qSave()
        end
        imgui.SameLine(0, 5 * MDS)
        if imgui.Button(T("skin_back"), V2(40 * MDS, 0)) then
            skinId = skinId - 1
            if skinId < 0 then skinId = 0 end
            skinIdInput[0] = skinId
            if skinId ~= 0 then
                setPlayerSkin(skinId)
            end
            qSave()
        end
        imgui.SameLine(0, 5 * MDS)
        if imgui.Button(T("skin_next"), V2(40 * MDS, 0)) then
            skinId = skinId + 1
            if skinId > 311 then skinId = 311 end
            skinIdInput[0] = skinId
            if skinId ~= 0 then
                setPlayerSkin(skinId)
            end
            qSave()
        end
    end
    imgui.Spacing()
    imgui.TextColored(ACENT, T("chat_off"))
    Sep()
    local vchat = TL(T("chat_off"), chatOff, "chat_off")
    if vchat ~= chatOff then
        chatOff = vchat
        if chatOff then
            for i = 1, 15 do sampAddChatMessage("", -1) end
        end
        qSave()
    end
    imgui.Spacing()
end
function DrawGeralT(wdl)
    if curT[0] == 1 then
        DrawInterfaceTeam(wdl)
        DrawGeralS1(wdl)
        DrawGeralS2(wdl)
        DrawGeralS3()
        DrawGeralS4()
        DrawGeralS5(wdl)
    elseif curT[0] == 2 then
        DrawCoresT()
    elseif curT[0] == 3 then
        DrawMirasT()
    elseif curT[0] == 4 then
        DrawExtraT(wdl)
        imgui.Spacing()
        DrawCreditosT()
    end
end
function DrawCoresT()
    imgui.TextColored(ACENT, T("c"))
    Sep()
    local vrgb = TL(T("rgb"), hudRGB[0], "rgb")
    if vrgb ~= hudRGB[0] then
        if vrgb then
            saveCurrentColors()
        else
            restoreSavedColors()
        end
        hudRGB[0] = vrgb
        qSave()
    end
    imgui.Spacing()
    if not hudRGB[0] then
        local function CRow(l, c)
            if imgui.ColorEdit4("##ce_"..l, c, imgui.ColorEditFlags.NoInputs + imgui.ColorEditFlags.AlphaBar) then
                saveCurrentColors()
                qSave()
            end
            imgui.SameLine(0, 12 * MDS)
            imgui.Text(l)
            imgui.Dummy(V2(1, 5 * MDS))
        end
        CRow(T("v"), clr.hp)
        CRow(T("a"), clr.ap)
        if not st2[0] then
            CRow(T("s"), clr.spr)
        end
        CRow(T("d"), clr.d)
        CRow(T("e"), clr.e)
        CRow(T("iv"), clr.ic)
        CRow(T("ia"), clr.ic2)
        CRow(T("m"), clr.m)
        if imgui.ColorEdit4("##cfhp", corFonteHP, imgui.ColorEditFlags.NoInputs + imgui.ColorEditFlags.AlphaBar) then
            cfg.clr.fhp = {corFonteHP[0], corFonteHP[1], corFonteHP[2], corFonteHP[3]}
            qSave()
        end
        imgui.SameLine(0, 12 * MDS)
        imgui.Text("Fonte Vida")
        imgui.Dummy(V2(1, 5 * MDS))
        if imgui.ColorEdit4("##cfap", corFonteAP, imgui.ColorEditFlags.NoInputs + imgui.ColorEditFlags.AlphaBar) then
            cfg.clr.fap = {corFonteAP[0], corFonteAP[1], corFonteAP[2], corFonteAP[3]}
            qSave()
        end
        imgui.SameLine(0, 12 * MDS)
        imgui.Text("Fonte Colete")
        imgui.Dummy(V2(1, 5 * MDS))
    else
        imgui.TextColored(TXT_M, "RGB Mode Ativado - Cores dinâmicas")
    end
end
function DrawMirasT()
    imgui.TextColored(ACENT, T("mi"))
    Sep()
    local vmi = TL(T("mi_at"), miraAtivada[0], "mi_at")
    if vmi ~= miraAtivada[0] then miraAtivada[0] = vmi; qSave() end
    if miraAtivada[0] then
        imgui.Spacing()
        local tipos = ffi.new("const char*[30]", {"Cruz Simples", "Cruz Dupla", "Circulo", "Quadrado", "X", "Ponto", "Alvo", "Diamante", "Estrela", "Triangulo", "Cruz Fina", "Cruz Grossa", "Circulo Duplo", "Quadrado Vazado", "X com Ponto", "Alvo Precisão", "Diamante Vazado", "Estrela 6 Pontas", "Triangulo Invertido", "Cruz Suíça", "Alvo Tático", "Ponto Duplo", "Círculo com Cruz", "Quadrado com Cruz", "X com Círculo", "Hexágono", "Octógono", "Círculo Segmentado", "Cruz Tática", "Alvo Avançado"})
        imgui.PushItemWidth(200 * MDS)
        if imgui.Combo(T("mi_t"), miraTipo, tipos, 30) then qSave() end
        if imgui.SliderFloat(T("mi_sz"), miraTamanho, 2, 50) then qSave() end
        if imgui.SliderFloat(T("mi_w"), miraLargura, 0.5, 10) then qSave() end
        imgui.PopItemWidth()
        if imgui.ColorEdit4("Cor da Mira", corMira, imgui.ColorEditFlags.NoInputs + imgui.ColorEditFlags.AlphaBar) then qSave() end
        imgui.Spacing()
        imgui.TextColored(ACENT, "Posição da Mira")
        Sep()
        if SI("Mira X", miraX, -1000, 1000) then qSave() end
        if SI("Mira Y", miraY, -1000, 1000) then qSave() end
        imgui.Spacing()
        local vmb = TL(T("mi_b"), miraBordaAtivada[0], "mi_b")
        if vmb ~= miraBordaAtivada[0] then miraBordaAtivada[0] = vmb; qSave() end
        if miraBordaAtivada[0] then
            imgui.PushItemWidth(200 * MDS)
            if imgui.SliderFloat(T("mi_bs"), miraTamanhoBorda, 0.5, 5) then qSave() end
            imgui.PopItemWidth()
            if imgui.ColorEdit4("Cor da Borda", corBordaMira, imgui.ColorEditFlags.NoInputs + imgui.ColorEditFlags.AlphaBar) then qSave() end
        end
    end
end
function DrawCreditosT()
    imgui.TextColored(ACENT, T("crd"))
    Sep()
    imgui.TextColored(TXT_M, T("atc"))
    imgui.TextColored(TXT_M, T("atv"))
end
local function DrawH(wp, ws)
    WDL:AddRectFilled(V2(wp.x, wp.y), V2(wp.x+ws.x, wp.y+42), u32(V4(0.06,0.06,0.06,0.75)), 0)
    WDL:AddRectFilled(V2(wp.x, wp.y+41), V2(wp.x+ws.x, wp.y+43), u32(V4(0.22,0.22,0.22,0.8)))
    local tit = T("t")
    local tsT = imgui.CalcTextSize(tit)
    WDL:AddText(V2(wp.x+15*MDS, wp.y+(42-tsT.y)*0.5), u32(V4(0.85,0.85,0.85,1)), tit)
end
local function DrawLang(wp, ws)
    -- Função desativada pois o seletor foi para o rodapé
end
local function DrawTB(wp, ws)
    local tabs = {T("g"), T("c"), T("mi"), T("cr")}
    local tW = (ws.x - 14 * MDS) / #tabs
    local tH = 26 * MDS
    local ty = 43 + 5 * MDS
    for i, n in ipairs(tabs) do
        local tx = wp.x + 7 * MDS + (i-1) * tW
        local sel = curT[0] == i
        local bgTab = sel and u32(V4(0.22,0.22,0.22,1)) or u32(V4(0.12,0.12,0.12,1))
        WDL:AddRectFilled(V2(tx, wp.y+ty), V2(tx + tW - 2, wp.y+ty+tH), bgTab, 8)
        if sel then
            WDL:AddRect(V2(tx, wp.y+ty), V2(tx + tW - 2, wp.y+ty+tH), u32(V4(0.35,0.35,0.35,0.8)), 8, 0, 1.2)
            WDL:AddRectFilled(V2(tx+10, wp.y+ty+tH-2), V2(tx + tW - 12, wp.y+ty+tH), u32(V4(0.65,0.65,0.65,1)), 2)
        end
        local ts = imgui.CalcTextSize(n)
        WDL:AddText(V2(tx + (tW - ts.x)*0.5, wp.y+ty + (tH - ts.y)*0.5),
            sel and u32(V4(0.88,0.88,0.88,1)) or u32(V4(0.42,0.42,0.42,1)), n)
        imgui.SetCursorPos(V2(tx - wp.x, ty))
        imgui.InvisibleButton("##tab"..i, V2(tW - 2, tH))
        if imgui.IsItemClicked() then curT[0] = i end
    end
end
local function DrawF(wp, fh, cw)
    local fy = wH - fh
    WDL:AddRectFilled(V2(wp.x, wp.y+fy), V2(wp.x + wW, wp.y + wH), u32(V4(0.06,0.06,0.06,1)), 0)
    WDL:AddRectFilled(V2(wp.x, wp.y+fy), V2(wp.x + wW, wp.y+fy+1), u32(V4(0.20,0.20,0.20,0.8)), 0)
    
    -- Botão Fechar
    imgui.SetCursorPos(V2(8 * MDS, fy + 6 * MDS))
    local ch = fh - 12 * MDS
    local cp = imgui.GetCursorScreenPos()
    WDL:AddRectFilled(cp, V2(cp.x + cw, cp.y + ch), u32(V4(0.20,0.20,0.20,0.9)), 8)
    WDL:AddRect(cp, V2(cp.x + cw, cp.y + ch), u32(V4(0.32,0.32,0.32,0.8)), 8, 0, 1)
    local cl = T("fch")
    local cls = imgui.CalcTextSize(cl)
    WDL:AddText(V2(cp.x + (cw - cls.x)*0.5, cp.y + (ch - cls.y)*0.5), u32(V4(0.75,0.75,0.75,1)), cl)
    imgui.InvisibleButton("##fc", V2(cw, ch))
    if imgui.IsItemClicked() then sets[0] = false end

    local langN = ffi.new("const char*[4]", {"BR", "EN", "ES", "AB"})
    local wSel = 100 * MDS
    imgui.SameLine(0, 10 * MDS)
    imgui.PushItemWidth(wSel)
    if imgui.Combo("##id_footer", idioma, langN, 4) then qSave() end
    imgui.PopItemWidth()
end
function main()
    local nomeCorreto = "HasleHud v6 by Shellder.lua"
    local caminhoAtual = thisScript().path
    local nomeAtual = thisScript().name
    if nomeAtual ~= nomeCorreto:gsub(".lua", "") then
        local dir = caminhoAtual:match("(.+[\\/])")
        local novoCaminho = dir .. nomeCorreto
        os.rename(caminhoAtual, novoCaminho)
    end
    while not isSampAvailable() do wait(0) end
    if not checkFont() then
        showUpdateScreen = true
    end
    if not oh[0] then displayHud(false) end
    sampRegisterChatCommand("shell", function()
        if mMode then
            mMode = false; mDrag = false; qSave()
        else
            exibir_menu[0] = not exibir_menu[0]
            if exibir_menu[0] then
                showSplash = true
                splashTimer = os.clock()
            end
        end
    end)
    sampRegisterChatCommand("chatt", function()
        chatOff = not chatOff
        if chatOff then
            for i = 1, 15 do sampAddChatMessage("", -1) end
        end
        qSave()
    end)
    sampAddChatMessage("Hasslehud by Shellder | Comando: /shell", -1)
    while true do wait(0) end
end
function onScriptTerminate(s, q)
    if s == thisScript() then
        if not oh[0] then displayHud(true) end
        saveCfg()
    end
end
icons85 = {
    hungry = "\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0D\x49\x48\x44\x52\x00\x00\x00\x30\x00\x00\x00\x30\x08\x06\x00\x00\x00\x57\x02\xF9\x87\x00\x00\x00\x04\x73\x42\x49\x54\x08\x08\x08\x08\x7C\x08\x64\x88\x00\x00\x00\x09\x70\x48\x59\x73\x00\x00\x02\x76\x00\x00\x02\x76\x01\xDA\x60\xE3\x4F\x00\x00\x00\x19\x74\x45\x58\x74\x53\x6F\x66\x74\x77\x61\x72\x65\x00\x77\x77\x77\x2E\x69\x6E\x6B\x73\x63\x61\x70\x65\x2E\x6F\x72\x67\x9B\xEE\x3C\x1A\x00\x00\x03\x05\x49\x44\x41\x54\x68\x81\xED\x98\x4D\x48\x15\x51\x18\x86\xBF\xD1\xFC\x41\xEC\x77\xE1\x22\x73\x53\x52\x90\x51\x44\x3F\xD0\x2A\x0C\x6A\xD1\xA2\xCC\x08\x77\x25\x14\x45\x42\x46\x10\xD5\xF2\x6E\xD2\xB6\x41\xAB\xDA\x45\x8B\x88\x36\x2E\xAC\x65\x0B\x49\xFA\xC1\x08\x22\x22\x31\xA8\x4C\x09\xDA\xA5\x49\x75\xCD\xA7\xC5\x9C\x5B\xC7\xEF\x9E\x99\x3B\x7A\xCF\xBD\x63\x74\x5F\x70\x31\xCE\x99\xF7\x7D\xBE\x33\xDF\x9C\x39\x73\x45\x2A\xAA\xA8\xA2\x7F\x5A\x41\xB9\x82\x80\x26\x11\x39\x2C\x22\x87\x44\xA4\x55\x44\x5A\xCC\xA9\x71\x11\x19\x13\x91\x01\x11\x19\x08\x82\xE0\x4B\xB9\x98\x12\x09\x68\x04\x32\xC0\x0C\x85\xF5\x03\xB8\x0E\xAC\x4A\x9B\x5B\x44\x44\x80\xDD\xC0\x44\x02\x70\xAD\x09\x60\x57\xDA\xF0\x9D\x09\x67\x3D\x4A\x33\x40\x47\x5A\xF0\xFB\x4D\x3B\x68\xBD\x07\x2E\x03\x5B\x09\x5B\xAB\x11\xD8\x06\x5C\x01\x3E\x44\x14\xB1\xB3\xDC\xF0\xDB\x81\x29\x05\x32\x07\x5C\x05\xEA\x63\xAE\xAB\x07\xFA\xCC\x58\x5B\x9F\x28\xD7\x33\x01\x2C\x07\x46\x15\xC0\x2F\xE0\xF8\x02\x3C\xBA\x1D\x45\xF4\x97\x92\xDB\x0E\xBF\xED\x68\x83\x0B\x8B\xF0\xE9\x77\xB4\x52\x53\x29\x98\xED\xD0\x76\x07\xFC\x8D\x45\x7A\xD5\x13\x3E\x2F\xB6\x4E\xF9\x66\xB6\x03\xAB\x81\x57\x2A\xF0\x35\x31\x3D\x9F\xC0\xF3\x92\xF2\x1B\xF0\xC9\xAC\xC3\x8E\x3A\xFA\x7E\x47\x91\x9E\x6D\xCA\x73\xD4\x17\xAF\x2B\xEC\xB1\x0A\xBB\xE3\xC1\xB3\x41\x79\x4E\xFB\x60\x75\x05\x6D\x50\x41\xB3\x40\xAB\x07\xDF\x44\x05\x54\x15\x1B\x24\x22\x47\xD4\xF1\xA3\x20\x08\xC6\x3C\xF8\xAE\x57\xC7\x93\xAE\x41\x3E\x0A\xD8\xAB\x8E\xEF\x7A\xF0\x14\x11\x39\xA8\x8E\xDF\x78\xF2\x9D\x2F\xF2\x97\xBB\x4D\x1E\x3C\x5D\xCB\xE8\x49\x1F\xBC\xAE\xB0\x59\x2B\xE4\x3B\x50\xED\xC1\xF3\x9A\x82\xFF\x46\x29\x5E\x64\x40\x8D\x0A\xFA\xEA\xC1\xF3\x04\xF9\x5B\x89\x3E\x1F\xBC\xAE\xB0\x2A\x33\xEB\xF6\x0A\xB4\xAC\x08\xBF\x8B\x0E\xF8\x71\x60\xA5\x4F\x6E\x1D\xFA\x56\x05\xEA\x87\x3A\x89\x47\x03\x70\x93\x7C\xCD\x50\xE4\x0B\xB1\x50\x70\x17\x90\x55\xA1\x0B\x5A\x85\x80\x3D\x8E\x49\x80\xF0\x7B\xA2\xB3\x54\xEC\x51\xF0\x39\xF5\x24\xB8\x7E\x33\x70\xDF\xD1\x32\x10\x7E\x4F\x1C\x48\x0B\x3E\xA7\x7B\xC0\x3E\xA0\xC6\x5C\x53\x4B\xF8\xF5\x75\x16\x18\x8A\x00\xC7\xDC\x8D\x36\x1F\x90\x75\x40\x2F\xF0\x04\x98\x2E\x00\x9B\x2D\x50\xD0\x4F\xF3\x17\xA7\x39\xE0\x16\xB0\xC2\x07\x7C\x33\xF0\xB2\x40\xA0\x0D\xDF\x45\xB2\xBB\x12\xA5\x17\x40\x7B\xD1\xE0\x06\xBE\x6E\xA1\xF0\xE6\xBA\xC5\x14\x30\x04\x1C\x03\x7C\x6C\x69\xFE\x14\xD0\x9B\x30\xFC\x23\xE6\x67\x8F\x08\xF8\x2C\xF0\x8E\xF9\xBD\x3E\x09\x3C\x20\xFC\x58\xD9\xE8\x0D\x5A\x15\xF0\x54\x81\x0C\x02\xEB\x62\xC6\x77\x44\xC0\xE7\xEE\x4C\x35\xB0\x1A\x0F\x5B\x8C\xA4\x05\xE8\x9F\x44\x22\xE1\xCD\xF8\xF1\x28\xF8\x54\x44\xB8\x71\xB2\xD5\x6C\xFE\xEF\x5C\xDA\x1C\xAD\x95\x1E\xBC\x01\x7A\xE6\x68\xA1\xD3\x66\x66\x33\x8E\xF1\xF3\x94\x02\x72\x1E\xD0\x79\xC7\xAC\xDA\xCA\xA8\xF1\x4B\xAE\x80\x3A\x60\x24\xA6\x80\x2C\xB0\xC5\x1A\xBF\xB4\x0A\x10\x11\x01\xD6\x00\xCF\x23\x0A\x38\x63\x8D\x6B\x51\xE7\x8A\xFE\x1E\xF0\x26\xC2\xBD\xCB\x39\xE0\xB3\x82\x1C\x24\x7C\x53\xB7\x00\x0F\xD5\xB9\xE1\xB4\xB9\xF3\x44\xF2\x17\x1B\x24\xD8\x85\x96\x5D\xE6\x4E\x44\xB5\x93\xAD\x11\xA0\x36\x6D\x5E\xA7\x88\x7F\x26\x72\xF0\x6B\xD3\xE6\x8C\x15\xE1\x96\xA0\x9B\xBF\xDB\xEB\x29\x60\x18\xE8\x59\xB2\x33\x5F\x51\x45\xFF\x81\x7E\x03\xAB\xB3\x0E\x45\xD4\x46\x0A\x43\x00\x00\x00\x00\x49\x45\x4E\x44\xAE\x42\x60\x82",
    heart = "\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0D\x49\x48\x44\x52\x00\x00\x00\x30\x00\x00\x00\x30\x08\x06\x00\x00\x00\x57\x02\xF9\x87\x00\x00\x00\x04\x73\x42\x49\x54\x08\x08\x08\x08\x7C\x08\x64\x88\x00\x00\x00\x09\x70\x48\x59\x73\x00\x00\x02\x76\x00\x00\x02\x76\x01\xDA\x60\xE3\x4F\x00\x00\x00\x19\x74\x45\x58\x74\x53\x6F\x66\x74\x77\x61\x72\x65\x00\x77\x77\x77\x2E\x69\x6E\x6B\x73\x63\x61\x70\x65\x2E\x6F\x72\x67\x9B\xEE\x3C\x1A\x00\x00\x03\x1B\x49\x44\x41\x54\x68\x81\xED\x98\x5F\x68\x4E\x71\x18\xC7\xBF\x3F\x63\x43\x61\xA4\x36\x12\x92\x22\x8A\x12\x91\x7F\xE1\x06\x43\xB2\xD6\x84\x1B\x2E\x44\xC9\x1D\x2D\x2E\x57\x6A\x51\xE2\x02\x77\x5C\x28\x24\x17\x2C\x6B\x97\xB8\x90\xF2\x6F\xB5\xB5\xE5\x62\x69\x51\x36\x5B\xDB\xD8\xAC\x66\xFF\x3E\x2E\xDE\xF3\xD6\xF1\x7B\xCF\x39\xEF\x79\xCF\x7B\xDE\x59\x39\x9F\x3A\x37\xE7\x7D\x9E\xEF\xF3\x7D\xCE\x79\xCE\x39\xBF\xF7\x27\x25\x24\x24\x24\x24\xFC\xCF\x98\x5C\x82\x81\x65\x92\x2A\x24\x6D\x91\xB4\x42\xD2\x2C\x49\xFD\x92\xDA\x25\xBD\x94\xD4\x68\x8C\x19\xB0\x72\x66\x4A\x3A\x28\x69\xB3\xA4\x35\x92\x8A\x25\x8D\x39\x39\x1F\x24\x3D\x33\xC6\xF4\xE7\xD5\x45\x08\xE3\x6B\x80\xA7\xC0\x38\xC1\x0C\x00\xD7\x81\x52\x60\x0E\x70\x15\xF8\x91\x25\x67\x18\xB8\x0F\x2C\x2D\x84\x71\x03\x5C\x06\x46\xB2\x98\xB0\xE9\x04\xBE\xE5\x98\x33\x04\x9C\x8E\xDB\xFC\xDD\x1C\x4D\xC4\x41\x5D\x2E\x3E\xA7\x07\xFC\x76\x45\xD2\x29\x8F\xF3\x4D\x92\x1E\x4B\x6A\x56\x6A\xFE\xCB\x25\x6D\x97\x74\x4C\xD2\x22\x1F\xAD\x3E\x49\x4F\x25\xBD\x96\xD4\x23\x69\x9E\xA4\x75\x92\x8E\x48\x5A\x69\xC5\xD6\x00\xBD\xC6\x98\x6B\x21\x7B\xC8\x04\xD8\x49\xE6\xBC\xF7\x01\x55\x80\xE7\x83\x0F\xCC\x02\x6A\xAD\xBC\x09\xA0\x0E\x98\xEB\x93\x53\x04\x9C\x04\x7E\x5A\xB5\x46\x81\x8D\x51\xCD\x1B\xE0\xAD\x25\xD8\x0D\xAC\x0A\x99\x5F\x01\xB4\x01\xAD\x40\x45\xC8\x9C\xD5\x40\xAF\x55\xF3\x66\xD4\x06\xB6\x7B\xCC\xE5\xEE\x48\x62\xB9\xD5\xDD\x0F\x8C\xB9\x6A\x9E\x89\x2A\x74\xC3\x32\x5F\x1F\xB3\xD7\xA0\xDA\x07\x80\x87\xC0\x39\xBF\x51\x0D\x23\x62\x8F\x4F\x55\x0C\xA6\xBE\x02\x5D\xC0\xD1\x7C\xB4\xC2\x16\xB4\x67\x71\x71\x1E\x5A\x95\xFC\xFD\x0D\x19\x0D\xFB\x5C\x44\x86\xCC\x8F\xD6\x8C\x88\x3A\xB6\xF9\x34\x83\xC0\x86\xB8\xFC\x66\xCC\x19\xD0\x25\xA9\xCC\x75\xAA\xCC\x18\xD3\x9D\x8B\x28\x50\x29\xE9\x91\x24\xBF\xE6\x3B\x25\x6D\x35\xC6\x74\x78\xE4\x2E\x94\xB4\xCC\x23\xE7\x8B\x31\xA6\x27\x4C\xF1\x8F\xD6\x15\xDB\x97\x25\xFE\x30\xD0\xE1\x71\xA5\xDD\x0C\x03\xBF\xAD\x73\x6D\xC0\x7C\x4B\xAB\x86\xBF\xDF\x44\x6E\x26\x80\x0B\x61\x1A\xB0\xDF\x42\xF7\xB2\xC4\x67\x5B\xF3\x8C\x38\x4D\x9E\x70\x4C\xB8\x79\x05\x94\x38\x3A\x17\xB3\xE8\x00\x64\x5F\xB5\x02\x7B\xAC\xA4\x51\x60\x6D\x40\xFC\xF7\x80\x82\x63\xB8\xDE\x3C\xC0\x25\x8F\x98\x47\xC0\x79\x8F\xE6\xBC\x68\x0B\xD3\xC0\x34\xA0\xC9\x4A\xFC\x84\x75\xBB\x5D\xF1\xD5\xA4\xBE\xD4\x36\x9D\x40\xB5\x47\xFC\x9D\x10\x46\x7F\x01\xEF\xAD\xA3\x91\xB0\x0F\x3F\xB0\xD7\x43\xF4\x1D\x50\x1A\x4A\x20\x58\x7B\x3A\xF0\x3C\xC0\xFC\x00\xB0\x2D\xDF\x3A\x22\xF5\xC7\xC4\xE6\x23\xB0\x20\x06\xED\xD9\xC0\x1B\x0F\xFD\x21\x60\x57\xDE\xE6\x9D\x22\x45\x40\x7D\x01\xEF\x44\x19\xF0\xD9\xA5\x3B\x08\xEC\x88\xC3\xBB\xBB\x48\x09\xD0\x50\xC0\x26\x96\x03\x4F\x48\xCD\xF7\xA6\x38\x3C\x7B\x15\x29\x06\x9E\x15\x6A\x9C\x26\x85\x38\xEF\x04\x50\x0E\x14\x15\xCA\x6B\x50\xE1\xBC\x9A\x20\xB5\x53\xF1\xC2\xC9\x69\x07\x96\x4C\x86\x6F\xDB\x44\xA4\x26\x1C\xF3\xEF\xAC\x9C\xDA\xC9\xF4\xEE\x36\x93\x53\x13\x3E\xE6\x01\xCE\xFE\x0B\xFF\x69\x53\xA1\x9A\x08\x30\xDF\x40\xC4\x65\x7A\x6C\x04\x34\xD1\x02\x1C\x72\x8E\x16\x1F\xF3\x25\xFF\xD4\x7C\x1A\xFC\x5F\xB1\x7E\x34\x92\xDA\x2B\x9D\x3A\xE4\xD0\xC4\xD4\x33\x9F\xC6\x69\xE2\x36\xDE\x4B\xE2\x71\xE0\x16\x50\x5C\x88\xDA\xD1\xB6\x2E\x7C\x00\xD6\x4B\x3A\xAE\xD4\x36\xBA\x24\xB5\x4A\x7A\x60\x8C\x69\x8E\xB3\x4E\x42\x42\x42\x42\x42\x42\x9A\x3F\xE2\x80\x6D\xD6\x1B\xDF\x0B\x0F\x00\x00\x00\x00\x49\x45\x4E\x44\xAE\x42\x60\x82",
    stamina = "\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0D\x49\x48\x44\x52\x00\x00\x00\x30\x00\x00\x00\x30\x08\x06\x00\x00\x00\x57\x02\xF9\x87\x00\x00\x00\x04\x73\x42\x49\x54\x08\x08\x08\x08\x7C\x08\x64\x88\x00\x00\x00\x09\x70\x48\x59\x73\x00\x00\x02\x76\x00\x00\x02\x76\x01\xDA\x60\xE3\x4F\x00\x00\x00\x19\x74\x45\x58\x74\x53\x6F\x66\x74\x77\x61\x72\x65\x00\x77\x77\x77\x2E\x69\x6E\x6B\x73\x63\x61\x70\x65\x2E\x6F\x72\x67\x9B\xEE\x3C\x1A\x00\x00\x04\x5E\x49\x44\x41\x54\x68\x81\xED\x98\x6F\x68\x56\x55\x1C\xC7\xBF\xC7\x36\x1D\xCD\x6C\xA8\xB5\x5C\xCD\xD2\xA0\xE6\x20\xCC\x5E\x94\xAC\x91\xC8\x0A\xCC\x89\x46\x94\x8D\x98\x88\x10\x95\xE9\xAB\x7A\x95\x4A\x81\x10\x8D\x1A\x95\xA5\xBE\x89\xE8\x45\xA5\xFD\x01\x21\x08\x0A\x8A\x4A\xA9\x2C\x23\xA5\x54\x24\x22\xCA\x59\xC9\xB4\x36\x37\xF3\xCF\xD6\x7C\x3E\xBD\xB8\xF7\x91\xF3\x9C\xE7\xDE\xFB\xDC\xFB\xDC\xFB\xEC\x79\xE1\xBE\x30\xB8\x3B\xE7\x7C\xBF\xBF\xEF\xEF\x9E\xFB\x9C\xF3\x3B\x47\x9A\xC0\x04\x2E\x6D\x98\x4A\x8A\x03\x4D\x92\xEA\xFC\x7F\x87\x8D\x31\x7F\x57\x32\x5E\x6A\x00\x33\x81\x75\xC0\xE7\xC0\x00\xC5\x38\x0E\x7C\x0A\xBC\x00\xDC\x52\x6D\xBF\x17\x01\x34\x00\x3D\xC0\xD9\x00\xD3\x61\xC8\x01\xEB\xAB\xED\x5D\x40\x3B\xD0\x97\xC0\xB8\x8D\x31\x60\x6E\x35\xCD\xAF\x04\x46\x1D\x53\x43\xC0\x9B\xC0\xFD\xC0\x4D\xC0\x0C\xA0\x0E\x98\x03\xDC\x07\xEC\x75\xC6\x77\x55\xCB\xFC\x52\xFF\x0D\xDA\x9F\xC4\x36\xE0\xAA\x90\xF1\xED\xC0\x27\x8E\xF9\x73\xE3\x3A\x03\xC0\x0D\xC0\x7A\xE0\x75\xFF\x4D\xE7\xF1\x2F\xB0\x22\x84\xB3\x00\xD8\x17\xF0\xF9\x8C\x00\xAB\xC6\xCB\xF8\x0C\xE0\x1D\xE7\x8D\xE7\x31\x0A\x74\x44\x70\xBF\x0A\xE0\x00\xF4\x02\x93\xC6\xC3\xFC\x8D\x44\xFF\x48\x9F\x2A\xC1\x7F\x37\x82\xBB\x1F\x58\x50\x49\xF3\x53\x81\x5F\x9D\xA0\x7F\xE2\x7D\x42\x1B\x80\x25\x31\x34\xEA\x81\xB5\xC0\x4E\x82\xF7\x86\x11\x60\x65\xA5\x12\xE8\x75\x82\x3D\x0F\xD4\xA6\xD0\x9B\x0C\xAC\xA0\x78\x25\x1A\x03\xDA\xB3\xF4\x9E\x7F\x73\xC3\x56\x90\x6D\xC0\x35\xC0\x01\xE0\x77\x60\x51\x4A\xFD\x07\x29\x5C\x08\x0E\x03\x89\x4B\x9B\x9A\x88\xBE\x0E\x49\x57\xF8\xCF\xE7\x24\x6D\x92\xD4\x2D\xE9\x56\xBF\x6D\x9D\xA4\xDD\x8E\xA9\x26\x49\xAB\x25\x8D\x49\x1A\xF0\xFF\x4E\xFB\xDD\x43\x92\x26\xF9\x9A\x57\x4A\x6A\x92\x74\x48\x52\x9B\xDF\xDF\x2A\xA9\x59\x52\x5F\x56\x09\xB4\x5A\xCF\xDF\x19\x63\x06\x81\x2F\x7D\x43\x53\x25\x7D\x1C\xC0\xD9\x25\xE9\x8E\x24\x06\x2C\x9C\x92\x74\x22\x29\x29\x2A\x81\x29\x8E\xB8\x8C\x31\x07\x81\xEB\x25\x4D\x33\xC6\x1C\x0D\xE0\x4C\x4B\x6A\xC0\xC7\x80\xA4\x2E\x63\xCC\xF9\xA4\xC4\xA8\x04\xFE\xB1\x9E\x5B\xF2\x0F\xC6\x98\x41\x49\x83\x21\x9C\x6E\x49\x1B\x24\x2D\x92\x34\x33\x64\x4C\x4E\x52\xBF\xA4\xE3\x92\x7E\x94\xB4\x47\xD2\x07\xC6\x98\x33\x31\x3D\xC7\x03\xDE\x0E\x6A\xA3\x33\x01\xD7\x00\x8B\x81\x1D\x14\x6F\x7E\xA3\xC0\xDA\x4C\xCD\x46\x98\xD8\xEF\xAC\xD7\x2F\x02\x0D\x09\x75\xE6\x01\x1F\x06\xAC\xFF\x5B\xCB\x59\x75\x12\x01\xB8\x93\xE2\x6A\xB3\x0F\x98\x55\x86\xD6\x6A\xE0\x94\xA3\xF5\x5A\x25\x7C\xBB\x81\x97\x3B\xFB\x01\xC0\xB3\x65\x6A\xB5\x50\x5C\x96\x6C\xCE\xDA\x73\x50\xE0\x46\x60\x8F\x15\xF4\x8D\x14\x5A\xB3\x81\x23\x96\xD6\x05\xE0\xDE\x2C\xFD\x86\x05\xDE\x62\x05\x7D\x2F\xA5\x56\x33\x70\xC2\xD2\x3B\x09\x5C\x9B\x95\xD7\xA0\x80\x8F\x02\xE7\xAD\x80\x6F\xC7\xE4\x19\xBC\x83\xCF\x76\xE0\x7D\x60\x33\xD0\xEC\xF7\x75\x50\xB8\x42\xED\xAC\x94\xF9\x4E\xBC\xD3\x96\x8D\x65\x31\x78\xD3\x81\xCF\x28\xC6\x08\xB0\xDC\x1F\xB3\xD1\x6A\xCF\x01\x6D\xA5\x74\xCB\x49\xE0\xB0\x15\x64\x18\x78\x38\x06\x67\x32\xF0\x7D\x80\xF9\x3C\xDE\xF2\xC7\xD5\x02\x07\xAD\xF6\x03\x40\x17\x59\x5D\xB9\x00\x73\x9D\xC0\xF7\xC4\xE4\x3D\xED\xF0\xF6\x01\x2F\x01\x7F\xE0\x55\xA0\x8B\xAD\xB1\x4B\x42\x92\xDC\x0B\xCC\x4B\x9B\xC0\x42\x4B\xF0\x02\x50\x63\xF5\x4D\x01\xEA\x42\x78\xF6\x21\x68\x97\xC3\x2B\x3A\x4F\x00\xDF\x86\x24\x71\x12\x68\x71\xC7\x27\x49\x60\x96\x23\xF8\x88\xDF\xBE\x0C\x38\x8D\xB7\x31\xB5\x39\x9C\x06\x87\x33\x3F\x46\x9C\xDB\x80\x5F\x80\xDF\x80\x63\x0E\xFF\x6B\xD2\xEC\xD8\xC0\x6E\x47\xF0\x10\x85\xAB\x47\x3F\xFE\xCA\x12\x92\x40\x6B\x94\x7E\x48\x4C\xF7\x13\xBC\x3D\x4D\x02\xF3\xF1\xAE\x4D\xA2\xF0\x13\xD0\x68\x71\xFE\xB2\xFA\x9E\x2C\x23\xA6\x01\x7E\xB6\x34\x22\x2F\x0E\xE2\x08\xB6\x51\x7C\xB8\x77\xCB\x8B\x23\xC0\x13\xC0\x47\xC0\x19\xAB\xFD\x18\x50\x5F\x46\x4C\xFB\x12\x2C\x7D\xB9\x01\xD4\x00\x77\xE3\x6D\x6A\x9D\x78\xD7\x85\xAF\x96\x98\x99\xB2\x66\x01\xB8\x8E\xC2\xF3\xF2\x9A\xD4\x09\x84\x04\x32\xC0\x2B\x31\x12\x08\xAC\xFF\x81\x87\xF0\x6A\xAC\x1F\x80\x4D\xC0\x5D\xC0\x1A\x0A\x67\xFB\x2C\x70\x75\x45\x12\xF0\x4D\x34\x39\x66\x77\x00\xAB\x80\x6E\xE0\x19\xE0\x71\xE0\xB2\x00\x5E\x4F\xCC\xD9\xDB\x58\x31\xF3\xBE\x11\x7B\xAF\xF8\x8F\x18\xD7\x85\x78\x17\xBD\x71\xB0\x85\x12\x4B\x68\xD4\x99\x38\x2E\xFA\x1D\xBD\x1E\x60\xBB\xBC\x6B\x14\x1B\xD3\x25\xCD\x91\xB4\x50\xD2\x63\x56\xFB\x51\x79\xB7\x19\x4B\x25\xCD\xF6\x79\xDF\x48\xDA\x6A\x8C\xF9\x22\x03\x7F\xA5\x41\xF8\x4E\x1A\x07\x0F\x8C\x8B\xC9\x12\x09\xDC\x4C\xE1\xDA\x1F\x07\x39\xE0\xB9\x6A\x7B\xBF\x08\xBC\xF2\xF9\x65\xBC\xFA\x25\x0C\xA3\x78\x45\x5A\x2F\x31\x4A\x8C\x38\xC8\xFC\x56\x00\xEF\x47\xD7\x28\xE9\x72\x49\xF5\x92\x6A\xE5\x5D\x8C\x0D\x4A\x1A\x32\xC6\xE4\xB2\x8E\x39\x81\x09\x5C\xCA\xF8\x1F\x8E\x91\x65\x7A\x13\x25\x23\x39\x00\x00\x00\x00\x49\x45\x4E\x44\xAE\x42\x60\x82",
    shield = "\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0D\x49\x48\x44\x52\x00\x00\x00\x30\x00\x00\x00\x30\x08\x06\x00\x00\x00\x57\x02\xF9\x87\x00\x00\x00\x04\x73\x42\x49\x54\x08\x08\x08\x08\x7C\x08\x64\x88\x00\x00\x00\x09\x70\x48\x59\x73\x00\x00\x02\x76\x00\x00\x02\x76\x01\xDA\x60\xE3\x4F\x00\x00\x00\x19\x74\x45\x58\x74\x53\x6F\x66\x74\x77\x61\x72\x65\x00\x77\x77\x77\x2E\x69\x6E\x6B\x73\x63\x61\x70\x65\x2E\x6F\x72\x67\x9B\xEE\x3C\x1A\x00\x00\x02\xAA\x49\x44\x41\x54\x68\x81\xED\x99\xCD\x4B\x15\x51\x18\xC6\x9F\x73\xD3\x52\xA1\x0C\x8D\x82\x92\x36\x7E\xB4\xE8\x63\xD1\x4E\xFA\x94\xA0\x55\x81\xCB\xA0\x16\x2A\xB4\xC8\xFA\x23\x5A\x14\xAD\x43\x97\x7D\x10\x24\x11\xD4\xA2\x45\x04\x41\x04\xB5\x34\x37\xB5\x8B\xAE\xC2\x05\x37\x49\xA4\x69\x69\xA1\xF7\xD7\x62\x46\x1A\x8F\x33\x77\xCE\xCC\x1D\x1D\x89\xF9\xC1\x85\x39\x77\xE6\x7D\xDE\xE7\x99\x39\x77\x2E\x73\x46\x2A\x28\x28\xC8\x1C\xA0\x04\x0C\x03\xD3\xFE\x67\x18\x28\xE5\xED\x2B\x16\xC0\x00\x17\x81\x09\xD6\xF3\x01\xB8\x00\x98\x3C\x8C\xB5\x00\x6D\x11\xFB\x9A\x81\xD3\xC0\x6D\x60\x32\xC4\xB8\x4D\x19\xB8\x05\x9C\x02\x9A\x22\x34\xDB\x80\x16\x17\x6F\xB1\x67\x03\xB8\x2A\x69\x44\xD2\x0E\x49\xDF\x25\x4D\x4B\x5A\xF2\x6B\xF7\x49\xDA\x2F\x29\xED\xF4\xA8\xFA\x7A\x5F\x25\x21\xA9\x49\x52\x87\xA4\xDD\x92\x7E\x4B\xBA\x61\x8C\xB9\x57\x6F\x80\x19\x49\x7B\x52\x98\x5B\x96\x34\xE6\x6F\x5F\x96\xD4\x90\x42\x63\xC6\x18\xB3\x37\x45\xDD\x3F\x1C\xA6\x84\xCD\x02\x70\x1F\xE8\x0C\x68\x74\x01\x0F\x80\x9F\x49\xC5\xE2\xFC\xB9\x5C\x81\x38\x91\x25\x49\x9F\x25\x4D\x48\x7A\x21\xE9\xB5\x31\x66\x31\x42\xAB\x59\xD2\x79\x49\xFD\x92\x8E\x4B\xEA\x91\x37\x6D\xA2\x0D\x1A\x53\xD3\x63\x9A\x00\x9D\xF2\xE6\xA8\x91\xF4\x4D\x52\xC5\x18\x53\x8D\xD3\x89\xD0\x2E\x49\x3A\x28\xA9\x5D\xDE\x6F\x60\x56\x52\x79\x8D\xC1\xAC\x03\xC4\x09\xD6\x4B\xD2\x7E\x5B\xFF\xCF\x25\x86\x22\x40\xDE\x14\x01\xF2\xA6\x08\x90\x37\x45\x80\xBC\x29\x02\xE4\x8D\x4B\x80\x5F\xC1\x81\xEB\x93\x52\x1A\x80\x9D\xD6\x57\xF3\x71\x35\x2E\x01\xE6\xAC\x71\x9A\x87\x1B\x57\x6C\xED\x1F\x71\x05\x2E\x01\xA6\xAC\xF1\x61\x67\x3B\xC9\x39\x62\x8D\xCB\xA1\x47\x05\x70\x09\xF0\xD1\x1A\x9F\x70\xB6\x93\x1C\x5B\xFB\x53\x5C\x81\x4B\x80\xF7\xD6\xB8\xDF\xD9\x4E\x72\x6C\xED\x77\x75\x2B\x02\xAD\xC0\x1F\xEB\x51\xF5\x4C\xDD\xC2\xEB\xFB\x9C\xB3\x7A\x2C\x01\xBB\xB2\x12\x7F\x6E\x89\xBF\xC9\x44\x78\x6D\x8F\xB7\x56\x8F\xA7\x59\x8A\x9F\x0C\x59\x30\xB8\x94\xA1\xFE\x95\x10\xFD\xDE\xAC\xF4\x57\x9B\xBC\xB2\x1A\xCC\x02\x5D\x19\xE8\x76\x03\x73\x96\xF6\xCB\x2C\x3C\xDB\x8D\x0E\xF9\xF3\x32\xC8\x17\xE0\x40\x1D\x9A\x1D\x78\x4B\x8D\x41\x16\x81\xEE\x2C\xBD\x07\x1B\x5E\x0F\xB9\xD4\x93\xC0\xD1\x14\x5A\xC7\x80\xA9\x10\xBD\x6B\x1B\xE1\x3D\xD8\xF8\x51\x48\xD3\x79\x60\x28\x81\xC6\x90\x5F\x63\xF3\x70\x23\xBD\xAF\x36\xDF\x06\x3C\x09\x69\x0E\xDE\xDD\xAA\xBD\x46\x6D\x2B\x30\x56\xA3\x36\xCD\xFA\x69\xAA\x10\x8D\xC0\x68\x84\x91\x0A\xD0\x17\x52\xD3\xE7\xEF\x0B\x63\x64\xD3\xCC\x5B\xA6\x06\xF1\x16\x73\x6D\x56\x80\x3B\x7E\xD0\x46\x7F\x7B\x25\xE4\xB8\x05\x60\x70\xD3\x8D\x5B\x21\x7A\x80\xF1\x88\x33\x3B\x1E\xB3\xAF\x27\x57\xF3\xAB\x00\x0D\xC0\x4D\x60\x39\xC2\x6C\x90\x2A\x70\x17\xD8\x9E\xB7\xEF\x75\x00\xBD\xD4\x7E\xCD\x54\x01\xCE\xE6\xED\xB3\x26\x78\x77\x9A\xC7\x21\xE6\x9F\x11\xF1\x9E\x6D\x4B\x02\x0C\xF8\x67\xBC\x02\x0C\xE4\xED\xA7\xA0\xE0\x7F\xE5\x2F\xC5\x5C\x14\xCA\x1A\xA4\xDE\x73\x00\x00\x00\x00\x49\x45\x4E\x44\xAE\x42\x60\x82",
    star = "\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0D\x49\x48\x44\x52\x00\x00\x00\x30\x00\x00\x00\x30\x08\x06\x00\x00\x00\x57\x02\xF9\x87\x00\x00\x00\x04\x73\x42\x49\x54\x08\x08\x08\x08\x7C\x08\x64\x88\x00\x00\x00\x09\x70\x48\x59\x73\x00\x00\x02\x76\x00\x00\x02\x76\x01\xDA\x60\xE3\x4F\x00\x00\x00\x19\x74\x45\x58\x74\x53\x6F\x66\x74\x77\x61\x72\x65\x00\x77\x77\x77\x2E\x69\x6E\x6B\x73\x63\x61\x70\x65\x2E\x6F\x72\x67\x9B\xEE\x3C\x1A\x00\x00\x02\x71\x49\x44\x41\x54\x68\x81\xED\x98\xBF\x6B\x14\x41\x18\x86\xDF\x51\x21\xA8\x39\x0C\x51\x1B\x2F\x81\x14\x01\xB1\x10\xD4\x48\x54\x14\x0B\xCB\x70\x95\x62\x27\x8A\x8D\x60\x40\xB4\xB2\x14\x02\xA2\x88\x60\x27\x16\x22\x16\x36\x16\x82\xFA\x07\xA4\x10\xD3\x88\x60\x61\x21\x44\xAD\xE2\x8F\x88\xA2\x68\x73\x46\xA3\x3E\x16\xBB\xC2\x25\x37\xBB\x99\x99\xBB\xC9\x04\xD9\xA7\xB9\x83\xE5\x9B\x79\xDE\x9D\x9D\xD9\x99\x95\x2A\x2A\x2A\x2A\xFE\x3B\x80\xD5\xC0\x04\xF0\x09\xF8\x00\x1C\x4F\xED\xE4\x05\x70\x89\x85\xCC\x01\x7D\xA9\xBD\x9C\x00\x86\x80\x79\xDA\xB9\x9C\xDA\xCD\x09\xE0\x9A\x45\x1E\xE0\x0B\xD0\x9B\xDA\xAF\x14\xA0\x06\x7C\x2D\x08\x00\x30\x9E\xDA\xB1\x14\xE0\x5C\x89\x3C\xC0\x4B\x60\x55\x6A\x4F\x2B\xF9\xCA\xF3\x7A\x89\x00\x00\x8D\xD4\xAE\x56\x80\x23\x0E\xF2\x00\x93\xA9\x5D\xAD\x00\x53\x8E\x01\x00\x76\xA4\xF6\x5D\x00\x30\xE2\x21\x0F\x70\xBB\x5B\x7D\x9B\x00\xD9\xCD\x92\xB6\x48\x1A\x90\x54\xCF\xFF\x37\x24\x8D\x78\x34\xF3\x43\xD2\x41\x49\xD3\xC6\x98\x6F\xBE\x0E\xAD\x14\x06\x00\x36\x4A\x1A\x97\xB4\x4D\xD2\xA0\x32\xD1\xBA\xA4\x9E\x4E\x3A\xB4\xD0\x94\x34\x23\x69\x56\xD2\x5B\x49\xEF\x24\xBD\x97\xF4\x26\xFF\x7D\x6E\x8C\x99\xF3\x6A\x11\x18\x00\x66\x3C\x1F\x8B\x58\x7C\x06\xC6\x8A\x5C\x8B\xD6\xE4\xF3\xCA\xEE\xFA\x4A\xA0\x5F\xD2\x5D\xB2\x47\xB7\x8D\xA2\x00\xFD\xF1\x7C\x82\xA8\x49\x3A\x64\xBB\x50\x14\xE0\x59\x3C\x97\x60\xAC\x93\xDD\x3A\x89\x81\x75\x92\xA6\x24\xED\x8C\x69\xE4\xC1\x53\x49\xFB\x8D\x31\xF3\x8B\x2F\x58\x47\xC0\x18\xD3\x54\x36\x64\x8F\x23\x8B\xB9\xF0\x44\xD2\x98\x4D\x7E\x49\x80\x1E\xE0\x5E\xC2\x15\xE8\x01\xB0\xB6\xA3\xF8\x64\x9B\xB4\x9B\x09\xE4\x6F\x01\x6B\x3A\x92\x6F\x09\x61\x80\xAB\xCB\x28\x3F\xD1\x15\x71\x4B\x90\xB3\xC0\xEF\xC8\xF2\x17\xA2\xC8\xB7\x84\x38\x89\xFD\xCC\xDB\x29\xBF\x80\x33\x51\xE5\x5B\x42\x3C\x8C\x10\x20\xE8\xD0\x1F\x7A\xBC\xDB\x15\x58\x57\x46\x3D\xA4\xC8\x3B\x00\xF0\x6F\x2B\xDD\x6D\xF6\x86\x14\x85\x8C\xC0\xBE\x90\x8E\x1C\x18\x06\x36\xF9\x16\x85\x04\xD8\x13\x50\xE3\x82\x91\x34\xEA\x5B\xB4\x92\x02\x04\xB5\xED\x15\x80\xEC\xCD\xE8\x73\x74\xF4\xC5\x7B\x1E\xF8\x8E\xC0\x76\x49\xEB\x7D\x3B\xF1\x60\x14\xCF\x0F\x5F\xBE\x01\x7C\xEE\xD0\xA4\xB2\xE5\xF6\x84\xB2\xF3\xAE\x0B\x7D\x92\xB6\x7A\x3A\xB9\x03\xDC\x70\x78\x21\xBD\x60\xD1\x19\x16\xE8\x05\x2E\x02\x4D\x87\xFA\x63\x31\x03\x5C\x2F\xE9\xF8\x23\x70\x9A\x92\x1D\x24\x30\x08\xDC\x01\xFE\x94\xB4\x73\x2A\x66\x80\x03\xB4\x6F\xE6\xBE\x03\x57\x80\x0D\x1E\xED\xEC\x06\x1E\x59\xE4\x7F\x02\xC3\xD1\x02\xE4\x9D\x37\x80\x69\xB2\xCD\xD7\x7D\x60\xA8\x83\xB6\x0E\x03\xAF\x72\xF9\x59\xE0\x68\x17\x55\x97\x0F\xA0\x96\xDA\xA1\xA2\xA2\x22\x11\x7F\x01\x4A\x39\xBD\xF2\x1A\xCF\x69\xD0\x00\x00\x00\x00\x49\x45\x4E\x44\xAE\x42\x60\x82"
}

exibir_menu = imgui.new.bool(false)
l_t, a_t = getScreenResolution()
C_B, C_P, C_C, C_F = 0xFFFFFFFF, 0xFF000000, 0xFF444444, 0xCC050505
t_a, u_t = 0, os.clock()
a_p, l_l, l_c, esp = 550, 280, 650, 25
l_tot = l_l + l_c + esp
j_x, j_y = (l_t - l_tot)/2, (a_t - a_p)/2
arr, a_ox, a_oy = false, 0, 0

function t_som()
    local p = PLAYER_PED
    if p then local x,y,z = getCharCoordinates(p); addOneOffSound(x,y,z,1149) end
end

imgui.OnFrame(function()
    imgui.GetIO().MouseDrawCursor = exibir_menu[0]
    return exibir_menu[0]
end, function()
    local io = imgui.GetIO()
    local ag = os.clock()
    t_a = t_a + (ag - u_t) * 1.5
    u_t = ag
    imgui.PushStyleColor(imgui.Col.WindowBg, imgui.ImVec4(0,0,0,0))
    imgui.SetNextWindowPos(imgui.ImVec2(j_x, j_y))
    imgui.SetNextWindowSize(imgui.ImVec2(l_tot, a_p))
    imgui.Begin("##menu", nil, 31)
    local DL = imgui.GetWindowDrawList()
    if imgui.IsWindowHovered() and io.MouseDown[0] and not arr then
        if not imgui.IsAnyItemActive() then arr = true; a_ox, a_oy = io.MousePos.x - j_x, io.MousePos.y - j_y end
    end
    if arr then
        if io.MouseDown[0] then j_x, j_y = io.MousePos.x - a_ox, io.MousePos.y - a_oy else arr = false end
    end
    local p_e, p_c = 0, l_l + esp
    DL:AddRectFilled(imgui.ImVec2(j_x+p_e, j_y), imgui.ImVec2(j_x+p_e+l_l, j_y+a_p), C_F)
    DL:AddRectFilled(imgui.ImVec2(j_x+p_c, j_y), imgui.ImVec2(j_x+p_c+l_c, j_y+a_p), C_F)
    local function d_b(ico, txt, dy, id, tid)
        imgui.SetCursorPos(imgui.ImVec2(p_e+(l_l-180)/2, dy))
        imgui.InvisibleButton(id, imgui.ImVec2(180, 45))
        if imgui.IsItemClicked() then t_som(); curT[0] = tid end
        local c = (curT[0] == tid) and C_C or (imgui.IsItemActive() and C_C or C_B)
        local pf = imgui.ImVec2(j_x+p_e+(l_l-180)/2, j_y+dy)
        DL:AddRectFilled(pf, imgui.ImVec2(pf.x+180, pf.y+45), c, 5)
        local tc = ico and (ico.." "..txt) or txt
        local ts = imgui.CalcTextSize(tc)
        DL:AddText(imgui.ImVec2(pf.x+(180-ts.x)/2, pf.y+(45-ts.y)/2), C_P, tc)
    end
    d_b(fa("user"), "PLAYER / GERAL", 80, "b1", 1)
    d_b(fa("palette"), "CORES", 145, "b2", 2)
    d_b(fa("crosshairs"), "MIRAS", 210, "b3", 3)
    d_b(fa("plus"), "EXTRAS", 275, "b4", 4)
    -- Botão Fechar (Tamanho Original)
    local bW_f = 180
    imgui.SetCursorPos(imgui.ImVec2(p_e+(l_l-bW_f)/2, 440))
    imgui.InvisibleButton("close", imgui.ImVec2(bW_f, 45))
    if imgui.IsItemClicked() then t_som(); exibir_menu[0] = false end
    local pf_c = imgui.ImVec2(j_x+p_e+(l_l-bW_f)/2, j_y+440)
    DL:AddRectFilled(pf_c, imgui.ImVec2(pf_c.x+bW_f, pf_c.y+45), imgui.IsItemActive() and C_C or C_B, 5)
    local ts_f = imgui.CalcTextSize(T("fch"))
    DL:AddText(imgui.ImVec2(pf_c.x+(bW_f-ts_f.x)/2, pf_c.y+(45-ts_f.y)/2), C_P, T("fch"))

    -- Botão Tradutor (Subido e com menu flutuante externo)
    local bW_l = 45
    local bH_l = 45
    -- Posicionado um pouco acima do botão fechar, no canto da barra lateral
    imgui.SetCursorPos(imgui.ImVec2(p_e+(l_l-bW_l)/2, 385)) 
    imgui.InvisibleButton("lang_toggle", imgui.ImVec2(bW_l, bH_l))
    if imgui.IsItemClicked() then t_som(); showLangMenu = not showLangMenu end
    local pf_l = imgui.ImVec2(j_x+p_e+(l_l-bW_l)/2, j_y+385)
    DL:AddRectFilled(pf_l, imgui.ImVec2(pf_l.x+bW_l, pf_l.y+bH_l), showLangMenu and C_C or C_B, 5)
    local ico_l = fa("language")
    local ts_l = imgui.CalcTextSize(ico_l)
    DL:AddText(imgui.ImVec2(pf_l.x+(bW_l-ts_l.x)/2, pf_l.y+(bH_l-ts_l.y)/2), C_P, ico_l)

    if showLangMenu then
        imgui.SetNextWindowPos(imgui.ImVec2(j_x - 70, j_y + 385))
        imgui.SetNextWindowSize(imgui.ImVec2(65, 200))
        imgui.PushStyleColor(imgui.Col.WindowBg, imgui.ImVec4(0.06, 0.06, 0.06, 0.95))
        imgui.Begin("##lang_float", nil, 16 + 64 + 128) -- NoTitle, NoResize, NoMove
        local langs = {"BR", "EN", "ES", "AB"}
        for i, l in ipairs(langs) do
            if imgui.Button(l.."##opt"..i, imgui.ImVec2(50, 40)) then
                idioma[0] = i - 1
                showLangMenu = false
                qSave()
            end
        end
        imgui.End()
        imgui.PopStyleColor()
    end
    imgui.SetCursorPos(imgui.ImVec2(p_c+15, 15))
    imgui.BeginChild("##cont", imgui.ImVec2(l_c-30, a_p-30), false)
    if curT[0]==1 then DrawGeralT(DL) elseif curT[0]==2 then DrawCoresT() elseif curT[0]==3 then DrawMirasT() elseif curT[0]==4 then DrawExtraT(DL); imgui.Spacing(); DrawCreditosT() end
    imgui.EndChild()
    local per, t_l = (l_tot*2)+(a_p*2), 120
    local v = (t_a*150)%per
    local function gP(p)
        p = p % per
        if p < l_tot then return j_x+p, j_y
        elseif p < l_tot+a_p then return j_x+l_tot, j_y+(p-l_tot)
        elseif p < (l_tot*2)+a_p then return j_x+l_tot-(p-l_tot-a_p), j_y+a_p
        else return j_x, j_y+a_p-(p-(l_tot*2)-a_p) end
    end
    local a_b = math.floor((math.sin(t_a*2)*0.5+0.5)*100+155)
    local c_r = bit.bor(bit.lshift(a_b, 24), 0xFFFFFF)
    for i=1,8 do
        local p1,p2 = v+((i-1)*15), v+(i*15)
        local x1,y1 = gP(p1); local x2,y2 = gP(p2)
        DL:AddLine(imgui.ImVec2(x1,y1), imgui.ImVec2(x2,y2), c_r, 3.0)
    end
    imgui.End(); imgui.PopStyleColor()
end)
