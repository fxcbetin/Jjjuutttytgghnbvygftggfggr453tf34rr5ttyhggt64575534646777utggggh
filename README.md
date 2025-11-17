local imgui = require "mimgui"
local faicons = require "fAwesome6"
local json = require "json"
local memory = require "memory"
local ffi = require "ffi"
local monethook = require "monethook"
local SAMemory = require "SAMemory"

SAMemory.require("CCamera")
local camera = SAMemory.camera
require("SAMemory.shared").require("RenderWare")

local configPadrao = {
    estiloHUD = 0,
    vidaX = 1479,
    vidaY = 92,
    coleteX = 1479,
    coleteY = 134,
    dinheiroX = 1551,
    dinheiroY = 175,
    vidaLargura = 50,
    vidaAltura = 29,
    coleteLargura = 50,
    coleteAltura = 29,
    tamanhoDinheiro = 63.0,
    tamanhoBordaDinheiro = 1.0,
    raioBorda = 6.8,
    corVida = {0.1, 0.3, 0.8, 1.0},
    corColete = {0.6, 0.6, 0.6, 1.0},
    corDinheiro = {1.0, 1.0, 1.0, 1.0},
    corBordaDinheiro = {0.0, 0.0, 0.0, 1.0},
    deslocamentoIconeX = -35,
    deslocamentoIconeY = 0,
    tamanhoIcone = 18.0,
    corFonte = {1.0, 1.0, 1.0, 1.0},
    tamanhoFonteHP = 13.0,
    tamanhoFonteAP = 13.0,
    tamanhoFonteDinheiro = 13.0,
    fonteHPX = 10,
    fonteHPY = 7,
    fonteAPX = 10,
    fonteAPY = 7,
    bordaAtivada = true,
    tamanhoBorda = 2.0,
    corBorda = {1.0, 1.0, 1.0, 1.0},
    miraAtivada = true,
    miraX = 0,
    miraY = 0,
    miraTamanho = 10.0,
    miraLargura = 2.0,
    corMira = {1.0, 1.0, 1.0, 1.0},
    miraBordaAtivada = true,
    miraTamanhoBorda = 1.0,
    corBordaMira = {0.0, 0.0, 0.0, 1.0},
    miraTipo = 1,
    tempoId = 12,
    climaId = 0,
    climaAtivo = false,
    fovAtivado = false,
    valorFov = 60,
    somenteNumerosHPAP = false,
    emojisAtivados = true,
    sensibilidade = 50,
    fixedZoom = 0,
    renderDistance = 600
}

local config = {}
for k, v in pairs(configPadrao) do
    config[k] = v
end

local estado = {
    vida = 100.0,
    colete = 100.0,
    dinheiro = 0,
    modoEdicao = imgui.new.bool(false),
    opacidadeTexto = imgui.new.float(1.0),
    direcaoFade = 1,
    indiceCor = 1,
    ultimoSalvamento = os.clock(),
    zoomLevel = 0,
    fovAtivo = false
}

local listaCores = {
    {0.8, 0.8, 0.8, 1.0},
    {1.0, 1.0, 1.0, 1.0},
    {0.6, 0.4, 0.2, 1.0},
    {0.0, 1.0, 0.0, 1.0},
    {1.0, 0.0, 0.0, 1.0},
    {1.0, 0.0, 1.0, 1.0}
}

local tiposMira = {
    "Cruz Simples",
    "Cruz Dupla", 
    "Circulo",
    "Quadrado",
    "X",
    "Ponto",
    "Alvo",
    "Diamante",
    "Estrela",
    "Triangulo",
    "Cruz Fina",
    "Cruz Grossa", 
    "Circulo Duplo",
    "Quadrado Vazado",
    "X com Ponto",
    "Alvo Precisão",
    "Diamante Vazado",
    "Estrela 6 Pontas",
    "Triangulo Invertido",
    "Cruz Suíça",
    "Alvo Tático",
    "Ponto Duplo",
    "Círculo com Cruz",
    "Quadrado com Cruz",
    "X com Círculo",
    "Hexágono",
    "Octógono",
    "Círculo Segmentado",
    "Cruz Tática",
    "Alvo Avançado"
}

local estilosHUD = {
    "Estilo Original",
    "Barrinha Fina"
}

local variaveis = {
    estiloHUD = imgui.new.int(config.estiloHUD),
    vidaX = imgui.new.int(config.vidaX),
    vidaY = imgui.new.int(config.vidaY),
    vidaLargura = imgui.new.int(config.vidaLargura),
    vidaAltura = imgui.new.int(config.vidaAltura),
    coleteX = imgui.new.int(config.coleteX),
    coleteY = imgui.new.int(config.coleteY),
    coleteLargura = imgui.new.int(config.coleteLargura),
    coleteAltura = imgui.new.int(config.coleteAltura),
    dinheiroX = imgui.new.int(config.dinheiroX),
    dinheiroY = imgui.new.int(config.dinheiroY),
    tamanhoDinheiro = imgui.new.float(config.tamanhoDinheiro),
    tamanhoBordaDinheiro = imgui.new.float(config.tamanhoBordaDinheiro),
    raioBorda = imgui.new.float(config.raioBorda),
    tamanhoFonteHP = imgui.new.float(config.tamanhoFonteHP),
    tamanhoFonteAP = imgui.new.float(config.tamanhoFonteAP),
    tamanhoFonteDinheiro = imgui.new.float(config.tamanhoFonteDinheiro),
    deslocamentoIconeX = imgui.new.int(config.deslocamentoIconeX),
    deslocamentoIconeY = imgui.new.int(config.deslocamentoIconeY),
    tamanhoIcone = imgui.new.float(config.tamanhoIcone),
    fonteHPX = imgui.new.int(config.fonteHPX),
    fonteHPY = imgui.new.int(config.fonteHPY),
    fonteAPX = imgui.new.int(config.fonteAPX),
    fonteAPY = imgui.new.int(config.fonteAPY),
    bordaAtivada = imgui.new.bool(config.bordaAtivada),
    tamanhoBorda = imgui.new.float(config.tamanhoBorda),
    corBorda = imgui.new.float[4](config.corBorda[1], config.corBorda[2], config.corBorda[3], config.corBorda[4]),
    miraAtivada = imgui.new.bool(config.miraAtivada),
    miraX = imgui.new.int(config.miraX),
    miraY = imgui.new.int(config.miraY),
    miraTamanho = imgui.new.float(config.miraTamanho),
    miraLargura = imgui.new.float(config.miraLargura),
    corMira = imgui.new.float[4](config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
    miraBordaAtivada = imgui.new.bool(config.miraBordaAtivada),
    miraTamanhoBorda = imgui.new.float(config.miraTamanhoBorda),
    corBordaMira = imgui.new.float[4](config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
    miraTipo = imgui.new.int(config.miraTipo),
    corVida = imgui.new.float[4](config.corVida[1], config.corVida[2], config.corVida[3], config.corVida[4]),
    corColete = imgui.new.float[4](config.corColete[1], config.corColete[2], config.corColete[3], config.corColete[4]),
    corDinheiro = imgui.new.float[4](config.corDinheiro[1], config.corDinheiro[2], config.corDinheiro[3], config.corDinheiro[4]),
    corBordaDinheiro = imgui.new.float[4](config.corBordaDinheiro[1], config.corBordaDinheiro[2], config.corBordaDinheiro[3], config.corBordaDinheiro[4]),
    corFonte = imgui.new.float[4](config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]),
    moverJuntos = imgui.new.bool(true),
    moverFontesJuntas = imgui.new.bool(true),
    tempoId = imgui.new.int(config.tempoId),
    climaId = imgui.new.int(config.climaId),
    climaAtivo = imgui.new.bool(config.climaAtivo),
    fovAtivado = imgui.new.bool(config.fovAtivado),
    valorFov = imgui.new.int(config.valorFov),
    somenteNumerosHPAP = imgui.new.bool(config.somenteNumerosHPAP),
    emojisAtivados = imgui.new.bool(config.emojisAtivados),
    sensibilidade = imgui.new.float(config.sensibilidade),
    fixedZoom = imgui.new.float(config.fixedZoom),
    renderDistance = imgui.new.float(config.renderDistance)
}

local gtasa = ffi.load("GTASA")
ffi.cdef("void _Z12AND_OpenLinkPKc(const char* link);")

local function abrirLink(link)
    gtasa._Z12AND_OpenLinkPKc(link)
end

local function rgba(r, g, b, a)
    return imgui.ColorConvertFloat4ToU32(imgui.ImVec4(r, g, b, a))
end

local function formatarDinheiro(valor)
    local formatado = tostring(valor)
    local k
    while true do
        formatado, k = string.gsub(formatado, "^(-?%d+)(%d%d%d)", '%1.%2')
        if k == 0 then break end
    end
    return formatado
end

local function salvarConfig()
    config.estiloHUD = variaveis.estiloHUD[0]
    config.vidaX = variaveis.vidaX[0]
    config.vidaY = variaveis.vidaY[0]
    config.vidaLargura = variaveis.vidaLargura[0]
    config.vidaAltura = variaveis.vidaAltura[0]
    config.coleteX = variaveis.coleteX[0]
    config.coleteY = variaveis.coleteY[0]
    config.coleteLargura = variaveis.coleteLargura[0]
    config.coleteAltura = variaveis.coleteAltura[0]
    config.dinheiroX = variaveis.dinheiroX[0]
    config.dinheiroY = variaveis.dinheiroY[0]
    config.tamanhoDinheiro = variaveis.tamanhoDinheiro[0]
    config.tamanhoBordaDinheiro = variaveis.tamanhoBordaDinheiro[0]
    config.raioBorda = variaveis.raioBorda[0]
    config.tamanhoFonteHP = variaveis.tamanhoFonteHP[0]
    config.tamanhoFonteAP = variaveis.tamanhoFonteAP[0]
    config.tamanhoFonteDinheiro = variaveis.tamanhoFonteDinheiro[0]
    config.deslocamentoIconeX = variaveis.deslocamentoIconeX[0]
    config.deslocamentoIconeY = variaveis.deslocamentoIconeY[0]
    config.tamanhoIcone = variaveis.tamanhoIcone[0]
    config.fonteHPX = variaveis.fonteHPX[0]
    config.fonteHPY = variaveis.fonteHPY[0]
    config.fonteAPX = variaveis.fonteAPX[0]
    config.fonteAPY = variaveis.fonteAPY[0]
    config.bordaAtivada = variaveis.bordaAtivada[0]
    config.tamanhoBorda = variaveis.tamanhoBorda[0]
    config.corBorda = {variaveis.corBorda[0], variaveis.corBorda[1], variaveis.corBorda[2], variaveis.corBorda[3]}
    config.miraAtivada = variaveis.miraAtivada[0]
    config.miraX = variaveis.miraX[0]
    config.miraY = variaveis.miraY[0]
    config.miraTamanho = variaveis.miraTamanho[0]
    config.miraLargura = variaveis.miraLargura[0]
    config.corMira = {variaveis.corMira[0], variaveis.corMira[1], variaveis.corMira[2], variaveis.corMira[3]}
    config.miraBordaAtivada = variaveis.miraBordaAtivada[0]
    config.miraTamanhoBorda = variaveis.miraTamanhoBorda[0]
    config.corBordaMira = {variaveis.corBordaMira[0], variaveis.corBordaMira[1], variaveis.corBordaMira[2], variaveis.corBordaMira[3]}
    config.miraTipo = variaveis.miraTipo[0]
    config.corVida = {variaveis.corVida[0], variaveis.corVida[1], variaveis.corVida[2], variaveis.corVida[3]}
    config.corColete = {variaveis.corColete[0], variaveis.corColete[1], variaveis.corColete[2], variaveis.corColete[3]}
    config.corDinheiro = {variaveis.corDinheiro[0], variaveis.corDinheiro[1], variaveis.corDinheiro[2], variaveis.corDinheiro[3]}
    config.corBordaDinheiro = {variaveis.corBordaDinheiro[0], variaveis.corBordaDinheiro[1], variaveis.corBordaDinheiro[2], variaveis.corBordaDinheiro[3]}
    config.corFonte = {variaveis.corFonte[0], variaveis.corFonte[1], variaveis.corFonte[2], variaveis.corFonte[3]}
    config.tempoId = variaveis.tempoId[0]
    config.climaId = variaveis.climaId[0]
    config.climaAtivo = variaveis.climaAtivo[0]
    config.fovAtivado = variaveis.fovAtivado[0]
    config.valorFov = variaveis.valorFov[0]
    config.somenteNumerosHPAP = variaveis.somenteNumerosHPAP[0]
    config.emojisAtivados = variaveis.emojisAtivados[0]
    config.sensibilidade = variaveis.sensibilidade[0]
    config.fixedZoom = variaveis.fixedZoom[0]
    config.renderDistance = variaveis.renderDistance[0]
    
    local caminhoArquivo = "/storage/emulated/0/.hudeditshellderwsw.json"
    local arquivo = io.open(caminhoArquivo, "w")
    if arquivo then
        arquivo:write(json.encode(config))
        arquivo:close()
    end
end

local function resetarConfiguracao()
    for k, v in pairs(configPadrao) do
        config[k] = v
    end
    
    variaveis.estiloHUD[0] = configPadrao.estiloHUD
    variaveis.vidaX[0] = configPadrao.vidaX
    variaveis.vidaY[0] = configPadrao.vidaY
    variaveis.vidaLargura[0] = configPadrao.vidaLargura
    variaveis.vidaAltura[0] = configPadrao.vidaAltura
    variaveis.coleteX[0] = configPadrao.coleteX
    variaveis.coleteY[0] = configPadrao.coleteY
    variaveis.coleteLargura[0] = configPadrao.coleteLargura
    variaveis.coleteAltura[0] = configPadrao.coleteAltura
    variaveis.dinheiroX[0] = configPadrao.dinheiroX
    variaveis.dinheiroY[0] = configPadrao.dinheiroY
    variaveis.tamanhoDinheiro[0] = configPadrao.tamanhoDinheiro
    variaveis.tamanhoBordaDinheiro[0] = configPadrao.tamanhoBordaDinheiro
    variaveis.raioBorda[0] = configPadrao.raioBorda
    variaveis.tamanhoFonteHP[0] = configPadrao.tamanhoFonteHP
    variaveis.tamanhoFonteAP[0] = configPadrao.tamanhoFonteAP
    variaveis.tamanhoFonteDinheiro[0] = configPadrao.tamanhoFonteDinheiro
    variaveis.deslocamentoIconeX[0] = configPadrao.deslocamentoIconeX
    variaveis.deslocamentoIconeY[0] = configPadrao.deslocamentoIconeY
    variaveis.tamanhoIcone[0] = configPadrao.tamanhoIcone
    variaveis.fonteHPX[0] = configPadrao.fonteHPX
    variaveis.fonteHPY[0] = configPadrao.fonteHPY
    variaveis.fonteAPX[0] = configPadrao.fonteAPX
    variaveis.fonteAPY[0] = configPadrao.fonteAPY
    variaveis.bordaAtivada[0] = configPadrao.bordaAtivada
    variaveis.tamanhoBorda[0] = configPadrao.tamanhoBorda
    variaveis.corBorda[0] = configPadrao.corBorda[1]
    variaveis.corBorda[1] = configPadrao.corBorda[2]
    variaveis.corBorda[2] = configPadrao.corBorda[3]
    variaveis.corBorda[3] = configPadrao.corBorda[4]
    variaveis.miraAtivada[0] = configPadrao.miraAtivada
    variaveis.miraX[0] = configPadrao.miraX
    variaveis.miraY[0] = configPadrao.miraY
    variaveis.miraTamanho[0] = configPadrao.miraTamanho
    variaveis.miraLargura[0] = configPadrao.miraLargura
    variaveis.corMira[0] = configPadrao.corMira[1]
    variaveis.corMira[1] = configPadrao.corMira[2]
    variaveis.corMira[2] = configPadrao.corMira[3]
    variaveis.corMira[3] = configPadrao.corMira[4]
    variaveis.miraBordaAtivada[0] = configPadrao.miraBordaAtivada
    variaveis.miraTamanhoBorda[0] = configPadrao.miraTamanhoBorda
    variaveis.corBordaMira[0] = configPadrao.corBordaMira[1]
    variaveis.corBordaMira[1] = configPadrao.corBordaMira[2]
    variaveis.corBordaMira[2] = configPadrao.corBordaMira[3]
    variaveis.corBordaMira[3] = configPadrao.corBordaMira[4]
    variaveis.miraTipo[0] = configPadrao.miraTipo
    variaveis.corVida[0] = configPadrao.corVida[1]
    variaveis.corVida[1] = configPadrao.corVida[2]
    variaveis.corVida[2] = configPadrao.corVida[3]
    variaveis.corVida[3] = configPadrao.corVida[4]
    variaveis.corColete[0] = configPadrao.corColete[1]
    variaveis.corColete[1] = configPadrao.corColete[2]
    variaveis.corColete[2] = configPadrao.corColete[3]
    variaveis.corColete[3] = configPadrao.corColete[4]
    variaveis.corDinheiro[0] = configPadrao.corDinheiro[1]
    variaveis.corDinheiro[1] = configPadrao.corDinheiro[2]
    variaveis.corDinheiro[2] = configPadrao.corDinheiro[3]
    variaveis.corDinheiro[3] = configPadrao.corDinheiro[4]
    variaveis.corBordaDinheiro[0] = configPadrao.corBordaDinheiro[1]
    variaveis.corBordaDinheiro[1] = configPadrao.corBordaDinheiro[2]
    variaveis.corBordaDinheiro[2] = configPadrao.corBordaDinheiro[3]
    variaveis.corBordaDinheiro[3] = configPadrao.corBordaDinheiro[4]
    variaveis.corFonte[0] = configPadrao.corFonte[1]
    variaveis.corFonte[1] = configPadrao.corFonte[2]
    variaveis.corFonte[2] = configPadrao.corFonte[3]
    variaveis.corFonte[3] = configPadrao.corFonte[4]
    variaveis.tempoId[0] = configPadrao.tempoId
    variaveis.climaId[0] = configPadrao.climaId
    variaveis.climaAtivo[0] = configPadrao.climaAtivo
    variaveis.fovAtivado[0] = configPadrao.fovAtivado
    variaveis.valorFov[0] = configPadrao.valorFov
    variaveis.somenteNumerosHPAP[0] = configPadrao.somenteNumerosHPAP
    variaveis.emojisAtivados[0] = configPadrao.emojisAtivados
    variaveis.sensibilidade[0] = configPadrao.sensibilidade
    variaveis.fixedZoom[0] = configPadrao.fixedZoom
    variaveis.renderDistance[0] = configPadrao.renderDistance
    
    salvarConfig()
    sampAddChatMessage("{FFFFFF}[HUD EDIT] {FF0000}Configuracoes resetadas!", -1)
end

local function carregarConfig()
    local caminhoArquivo = "/storage/emulated/0/.hudeditshellderwsw.json"
    if doesFileExist(caminhoArquivo) then
        local arquivo = io.open(caminhoArquivo, "r")
        if arquivo then
            local conteudo = arquivo:read("*a")
            arquivo:close()
            local salvo = json.decode(conteudo)
            if salvo then
                for k, v in pairs(configPadrao) do
                    config[k] = salvo[k] or v
                end
                
                variaveis.estiloHUD[0] = salvo.estiloHUD or configPadrao.estiloHUD
                variaveis.vidaX[0] = salvo.vidaX or configPadrao.vidaX
                variaveis.vidaY[0] = salvo.vidaY or configPadrao.vidaY
                variaveis.vidaLargura[0] = salvo.vidaLargura or configPadrao.vidaLargura
                variaveis.vidaAltura[0] = salvo.vidaAltura or configPadrao.vidaAltura
                variaveis.coleteX[0] = salvo.coleteX or configPadrao.coleteX
                variaveis.coleteY[0] = salvo.coleteY or configPadrao.coleteY
                variaveis.coleteLargura[0] = salvo.coleteLargura or configPadrao.coleteLargura
                variaveis.coleteAltura[0] = salvo.coleteAltura or configPadrao.coleteAltura
                variaveis.dinheiroX[0] = salvo.dinheiroX or configPadrao.dinheiroX
                variaveis.dinheiroY[0] = salvo.dinheiroY or configPadrao.dinheiroY
                variaveis.tamanhoDinheiro[0] = salvo.tamanhoDinheiro or configPadrao.tamanhoDinheiro
                variaveis.tamanhoBordaDinheiro[0] = salvo.tamanhoBordaDinheiro or configPadrao.tamanhoBordaDinheiro
                variaveis.raioBorda[0] = salvo.raioBorda or configPadrao.raioBorda
                variaveis.tamanhoFonteHP[0] = salvo.tamanhoFonteHP or configPadrao.tamanhoFonteHP
                variaveis.tamanhoFonteAP[0] = salvo.tamanhoFonteAP or configPadrao.tamanhoFonteAP
                variaveis.tamanhoFonteDinheiro[0] = salvo.tamanhoFonteDinheiro or configPadrao.tamanhoFonteDinheiro
                variaveis.deslocamentoIconeX[0] = salvo.deslocamentoIconeX or configPadrao.deslocamentoIconeX
                variaveis.deslocamentoIconeY[0] = salvo.deslocamentoIconeY or configPadrao.deslocamentoIconeY
                variaveis.tamanhoIcone[0] = salvo.tamanhoIcone or configPadrao.tamanhoIcone
                variaveis.fonteHPX[0] = salvo.fonteHPX or configPadrao.fonteHPX
                variaveis.fonteHPY[0] = salvo.fonteHPY or configPadrao.fonteHPY
                variaveis.fonteAPX[0] = salvo.fonteAPX or configPadrao.fonteAPX
                variaveis.fonteAPY[0] = salvo.fonteAPY or configPadrao.fonteAPY
                variaveis.bordaAtivada[0] = salvo.bordaAtivada ~= nil and salvo.bordaAtivada or configPadrao.bordaAtivada
                variaveis.tamanhoBorda[0] = salvo.tamanhoBorda or configPadrao.tamanhoBorda
                variaveis.miraAtivada[0] = salvo.miraAtivada ~= nil and salvo.miraAtivada or configPadrao.miraAtivada
                variaveis.miraX[0] = salvo.miraX or configPadrao.miraX
                variaveis.miraY[0] = salvo.miraY or configPadrao.miraY
                variaveis.miraTamanho[0] = salvo.miraTamanho or configPadrao.miraTamanho
                variaveis.miraLargura[0] = salvo.miraLargura or configPadrao.miraLargura
                variaveis.miraBordaAtivada[0] = salvo.miraBordaAtivada ~= nil and salvo.miraBordaAtivada or configPadrao.miraBordaAtivada
                variaveis.miraTamanhoBorda[0] = salvo.miraTamanhoBorda or configPadrao.miraTamanhoBorda
                variaveis.miraTipo[0] = salvo.miraTipo or configPadrao.miraTipo
                variaveis.tempoId[0] = salvo.tempoId or configPadrao.tempoId
                variaveis.climaId[0] = salvo.climaId or configPadrao.climaId
                variaveis.climaAtivo[0] = salvo.climaAtivo ~= nil and salvo.climaAtivo or configPadrao.climaAtivo
                variaveis.fovAtivado[0] = salvo.fovAtivado ~= nil and salvo.fovAtivado or configPadrao.fovAtivado
                variaveis.valorFov[0] = salvo.valorFov or configPadrao.valorFov
                variaveis.somenteNumerosHPAP[0] = salvo.somenteNumerosHPAP ~= nil and salvo.somenteNumerosHPAP or configPadrao.somenteNumerosHPAP
                variaveis.emojisAtivados[0] = salvo.emojisAtivados ~= nil and salvo.emojisAtivados or configPadrao.emojisAtivados
                variaveis.sensibilidade[0] = salvo.sensibilidade or configPadrao.sensibilidade
                variaveis.fixedZoom[0] = salvo.fixedZoom or configPadrao.fixedZoom
                variaveis.renderDistance[0] = salvo.renderDistance or configPadrao.renderDistance
                
                if salvo.corVida then
                    variaveis.corVida[0] = salvo.corVida[1] or configPadrao.corVida[1]
                    variaveis.corVida[1] = salvo.corVida[2] or configPadrao.corVida[2]
                    variaveis.corVida[2] = salvo.corVida[3] or configPadrao.corVida[3]
                    variaveis.corVida[3] = salvo.corVida[4] or configPadrao.corVida[4]
                end
                
                if salvo.corColete then
                    variaveis.corColete[0] = salvo.corColete[1] or configPadrao.corColete[1]
                    variaveis.corColete[1] = salvo.corColete[2] or configPadrao.corColete[2]
                    variaveis.corColete[2] = salvo.corColete[3] or configPadrao.corColete[3]
                    variaveis.corColete[3] = salvo.corColete[4] or configPadrao.corColete[4]
                end
                
                if salvo.corDinheiro then
                    variaveis.corDinheiro[0] = salvo.corDinheiro[1] or configPadrao.corDinheiro[1]
                    variaveis.corDinheiro[1] = salvo.corDinheiro[2] or configPadrao.corDinheiro[2]
                    variaveis.corDinheiro[2] = salvo.corDinheiro[3] or configPadrao.corDinheiro[3]
                    variaveis.corDinheiro[3] = salvo.corDinheiro[4] or configPadrao.corDinheiro[4]
                end
                
                if salvo.corBordaDinheiro then
                    variaveis.corBordaDinheiro[0] = salvo.corBordaDinheiro[1] or configPadrao.corBordaDinheiro[1]
                    variaveis.corBordaDinheiro[1] = salvo.corBordaDinheiro[2] or configPadrao.corBordaDinheiro[2]
                    variaveis.corBordaDinheiro[2] = salvo.corBordaDinheiro[3] or configPadrao.corBordaDinheiro[3]
                    variaveis.corBordaDinheiro[3] = salvo.corBordaDinheiro[4] or configPadrao.corBordaDinheiro[4]
                end
                
                if salvo.corFonte then
                    variaveis.corFonte[0] = salvo.corFonte[1] or configPadrao.corFonte[1]
                    variaveis.corFonte[1] = salvo.corFonte[2] or configPadrao.corFonte[2]
                    variaveis.corFonte[2] = salvo.corFonte[3] or configPadrao.corFonte[3]
                    variaveis.corFonte[3] = salvo.corFonte[4] or configPadrao.corFonte[4]
                end
                
                if salvo.corBorda then
                    variaveis.corBorda[0] = salvo.corBorda[1] or configPadrao.corBorda[1]
                    variaveis.corBorda[1] = salvo.corBorda[2] or configPadrao.corBorda[2]
                    variaveis.corBorda[2] = salvo.corBorda[3] or configPadrao.corBorda[3]
                    variaveis.corBorda[3] = salvo.corBorda[4] or configPadrao.corBorda[4]
                end
                
                if salvo.corMira then
                    variaveis.corMira[0] = salvo.corMira[1] or configPadrao.corMira[1]
                    variaveis.corMira[1] = salvo.corMira[2] or configPadrao.corMira[2]
                    variaveis.corMira[2] = salvo.corMira[3] or configPadrao.corMira[3]
                    variaveis.corMira[3] = salvo.corMira[4] or configPadrao.corMira[4]
                end
                
                if salvo.corBordaMira then
                    variaveis.corBordaMira[0] = salvo.corBordaMira[1] or configPadrao.corBordaMira[1]
                    variaveis.corBordaMira[1] = salvo.corBordaMira[2] or configPadrao.corBordaMira[2]
                    variaveis.corBordaMira[2] = salvo.corBordaMira[3] or configPadrao.corBordaMira[3]
                    variaveis.corBordaMira[3] = salvo.corBordaMira[4] or configPadrao.corBordaMira[4]
                end
            end
        end
    end
end

local function verificarMudancasESalvar()
    salvarConfig()
end

local sensibilidadeEndereco = MONET_GTASA_BASE + 6987568

local function aplicarSensibilidade(valor)
    memory.setfloat(sensibilidadeEndereco, 0.001 + valor / 3000)
end

local function setCameraRenderDistance(distancia)
    camera.pRwCamera.farplane = distancia
end

local function setCameraZoom(zoom)
    if isCurrentCharWeapon(PLAYER_PED, 34) and isCharPlayingAnim(PLAYER_PED, "gun_stand") or zoom == 0 then
        camera.bUseScriptZoomValuePed = false
    else
        camera.bUseScriptZoomValuePed = true
    end
    camera.fPedZoomValueScript = zoom
end

local fonteDinheiro, fonteHP, fonteAP

imgui.OnInitialize(function()
    local conf = imgui.ImFontConfig()
    conf.MergeMode = true
    conf.PixelSnapH = true
    local intervalosIcones = imgui.new.ImWchar[3](faicons.min_range, faicons.max_range, 0)
    imgui.GetIO().Fonts:AddFontFromMemoryCompressedBase85TTF(
        faicons.get_font_data_base85("Solid"), 22, conf, intervalosIcones
    )
    fonteDinheiro = imgui.GetIO().Fonts:AddFontDefault()
    fonteHP = imgui.GetIO().Fonts:AddFontDefault()
    fonteAP = imgui.GetIO().Fonts:AddFontDefault()
    imgui.GetIO().Fonts:Build()
end)

local function limitarBarra(percentual, larguraMaxima)
    return math.min(percentual * larguraMaxima, larguraMaxima)
end

local function desenharHUDOriginal(desenho, pos)
    local vidaMaxima = 100
    local coleteMaximo = 100
    local percentualVida = math.min(estado.vida / vidaMaxima, 1.0)
    local percentualColete = math.min(estado.colete / coleteMaximo, 1.0)
    
    local larguraBarraVida = limitarBarra(percentualVida, config.vidaLargura)
    local larguraBarraColete = limitarBarra(percentualColete, config.coleteLargura)
    
    local fundoVida = imgui.ImVec2(pos.x + config.vidaX, pos.y + config.vidaY)
    local fimVida = imgui.ImVec2(pos.x + config.vidaX + config.vidaLargura, pos.y + config.vidaY + config.vidaAltura)
    local frenteVida = imgui.ImVec2(pos.x + config.vidaX + larguraBarraVida, pos.y + config.vidaY + config.vidaAltura)
    
    desenho:AddRectFilled(fundoVida, fimVida, rgba(0.1, 0.1, 0.1, 0.8), config.raioBorda)
    if larguraBarraVida > 0 then
        desenho:AddRectFilled(fundoVida, frenteVida, rgba(config.corVida[1], config.corVida[2], config.corVida[3], config.corVida[4]), config.raioBorda)
    end
    if config.bordaAtivada and config.tamanhoBorda > 0 then
        desenho:AddRect(fundoVida, fimVida, 
                     rgba(config.corBorda[1], config.corBorda[2], config.corBorda[3], config.corBorda[4]), 
                     config.raioBorda, 0, config.tamanhoBorda)
    end
    
    imgui.SetWindowFontScale(config.tamanhoFonteHP / 13.0)
    local textoVida = string.format("%d", math.floor(estado.vida))
    if not config.somenteNumerosHPAP then
        textoVida = textoVida .. " HP"
    end
    local posTextoVida = imgui.ImVec2(
        pos.x + config.vidaX + config.fonteHPX, 
        pos.y + config.vidaY + config.fonteHPY
    )
    desenho:AddText(posTextoVida, rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), textoVida)
    imgui.SetWindowFontScale(1.0)
    
    if config.emojisAtivados then
        imgui.SetWindowFontScale(config.tamanhoIcone / 22.0)
        desenho:AddText(imgui.ImVec2(pos.x + config.vidaX + config.deslocamentoIconeX, pos.y + config.vidaY + 5 + config.deslocamentoIconeY), rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), faicons.HEART_PULSE)
        imgui.SetWindowFontScale(1.0)
    end
    
    local fundoColete = imgui.ImVec2(pos.x + config.coleteX, pos.y + config.coleteY)
    local fimColete = imgui.ImVec2(pos.x + config.coleteX + config.coleteLargura, pos.y + config.coleteY + config.coleteAltura)
    local frenteColete = imgui.ImVec2(pos.x + config.coleteX + larguraBarraColete, pos.y + config.coleteY + config.coleteAltura)
    
    desenho:AddRectFilled(fundoColete, fimColete, rgba(0.1, 0.1, 0.1, 0.8), config.raioBorda)
    if larguraBarraColete > 0 then
        desenho:AddRectFilled(fundoColete, frenteColete, rgba(config.corColete[1], config.corColete[2], config.corColete[3], config.corColete[4]), config.raioBorda)
    end
    if config.bordaAtivada and config.tamanhoBorda > 0 then
        desenho:AddRect(fundoColete, fimColete, 
                     rgba(config.corBorda[1], config.corBorda[2], config.corBorda[3], config.corBorda[4]), 
                     config.raioBorda, 0, config.tamanhoBorda)
    end
    
    imgui.SetWindowFontScale(config.tamanhoFonteAP / 13.0)
    local textoColete = string.format("%d", math.floor(estado.colete))
    if not config.somenteNumerosHPAP then
        textoColete = textoColete .. " AP"
    end
    local posTextoColete = imgui.ImVec2(
        pos.x + config.coleteX + config.fonteAPX, 
        pos.y + config.coleteY + config.fonteAPY
    )
    desenho:AddText(posTextoColete, rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), textoColete)
    imgui.SetWindowFontScale(1.0)
    
    if config.emojisAtivados then
        imgui.SetWindowFontScale(config.tamanhoIcone / 22.0)
        desenho:AddText(imgui.ImVec2(pos.x + config.coleteX + config.deslocamentoIconeX, pos.y + config.coleteY + 5 + config.deslocamentoIconeY), rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), faicons.VEST)
        imgui.SetWindowFontScale(1.0)
    end
    
    imgui.SetWindowFontScale(config.tamanhoFonteDinheiro / 13.0)
    local textoDinheiro = "$" .. formatarDinheiro(estado.dinheiro)
    local posTexto = imgui.ImVec2(pos.x + config.dinheiroX, pos.y + config.dinheiroY)
    for x = -config.tamanhoBordaDinheiro, config.tamanhoBordaDinheiro, config.tamanhoBordaDinheiro do
        for y = -config.tamanhoBordaDinheiro, config.tamanhoBordaDinheiro, config.tamanhoBordaDinheiro do
            if x ~= 0 or y ~= 0 then
                desenho:AddText(imgui.ImVec2(posTexto.x + x, posTexto.y + y),
                    rgba(config.corBordaDinheiro[1], config.corBordaDinheiro[2], config.corBordaDinheiro[3], config.corBordaDinheiro[4]),
                    textoDinheiro)
            end
        end
    end
    desenho:AddText(posTexto, rgba(config.corDinheiro[1], config.corDinheiro[2], config.corDinheiro[3], config.corDinheiro[4]), textoDinheiro)
    imgui.SetWindowFontScale(1.0)
end

local function desenharHUDBarrinhaFina(desenho, pos)
    local vidaMaxima = 100
    local coleteMaximo = 100
    local percentualVida = math.min(estado.vida / vidaMaxima, 1.0)
    local percentualColete = math.min(estado.colete / coleteMaximo, 1.0)
    local larguraBarra = 150
    local alturaBarra = 8
    local espacamento = 15
    local x = config.vidaX
    local yVida = config.vidaY
    local yColete = yVida + alturaBarra + espacamento
    
    local larguraBarraVida = limitarBarra(percentualVida, larguraBarra)
    local larguraBarraColete = limitarBarra(percentualColete, larguraBarra)
    
    local fundoVida = imgui.ImVec2(pos.x + x, pos.y + yVida)
    local fimVida = imgui.ImVec2(pos.x + x + larguraBarra, pos.y + yVida + alturaBarra)
    local frenteVida = imgui.ImVec2(pos.x + x + larguraBarraVida, pos.y + yVida + alturaBarra)
    local fundoColete = imgui.ImVec2(pos.x + x, pos.y + yColete)
    local fimColete = imgui.ImVec2(pos.x + x + larguraBarra, pos.y + yColete + alturaBarra)
    local frenteColete = imgui.ImVec2(pos.x + x + larguraBarraColete, pos.y + yColete + alturaBarra)
    
    desenho:AddRectFilled(fundoVida, fimVida, rgba(0.1, 0.1, 0.1, 0.8), 2.0)
    if percentualVida > 0 then
        desenho:AddRectFilled(fundoVida, frenteVida, rgba(config.corVida[1], config.corVida[2], config.corVida[3], config.corVida[4]), 2.0)
    end
    desenho:AddRectFilled(fundoColete, fimColete, rgba(0.1, 0.1, 0.1, 0.8), 2.0)
    if percentualColete > 0 then
        desenho:AddRectFilled(fundoColete, frenteColete, rgba(config.corColete[1], config.corColete[2], config.corColete[3], config.corColete[4]), 2.0)
    end
    if config.bordaAtivada and config.tamanhoBorda > 0 then
        desenho:AddRect(fundoVida, fimVida, 
                     rgba(config.corBorda[1], config.corBorda[2], config.corBorda[3], config.corBorda[4]), 
                     2.0, 0, config.tamanhoBorda)
        desenho:AddRect(fundoColete, fimColete, 
                     rgba(config.corBorda[1], config.corBorda[2], config.corBorda[3], config.corBorda[4]), 
                     2.0, 0, config.tamanhoBorda)
    end
    
    imgui.SetWindowFontScale(config.tamanhoFonteHP / 13.0)
    local textoVida = string.format("%d", math.floor(estado.vida))
    if not config.somenteNumerosHPAP then
        textoVida = textoVida .. " HP"
    end
    local posTextoVida = imgui.ImVec2(
        pos.x + x + larguraBarra + 10 + config.fonteHPX, 
        pos.y + yVida + config.fonteHPY
    )
    desenho:AddText(posTextoVida, 
                   rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), 
                   textoVida)
    
    imgui.SetWindowFontScale(config.tamanhoFonteAP / 13.0)
    local textoColete = string.format("%d", math.floor(estado.colete))
    if not config.somenteNumerosHPAP then
        textoColete = textoColete .. " AP"
    end
    local posTextoColete = imgui.ImVec2(
        pos.x + x + larguraBarra + 10 + config.fonteAPX, 
        pos.y + yColete + config.fonteAPY
    )
    desenho:AddText(posTextoColete, 
                   rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), 
                   textoColete)
    imgui.SetWindowFontScale(1.0)
    
    if config.emojisAtivados then
        imgui.SetWindowFontScale(config.tamanhoIcone / 22.0)
        desenho:AddText(imgui.ImVec2(pos.x + x - 25 + config.deslocamentoIconeX, pos.y + yVida + config.deslocamentoIconeY), 
                       rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), 
                       faicons.HEART_PULSE)
        desenho:AddText(imgui.ImVec2(pos.x + x - 25 + config.deslocamentoIconeX, pos.y + yColete + config.deslocamentoIconeY), 
                       rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), 
                       faicons.VEST)
        imgui.SetWindowFontScale(1.0)
    end
    
    imgui.SetWindowFontScale(config.tamanhoFonteDinheiro / 13.0)
    local textoDinheiro = "$" .. formatarDinheiro(estado.dinheiro)
    local posTexto = imgui.ImVec2(pos.x + config.dinheiroX, pos.y + config.dinheiroY)
    for x = -config.tamanhoBordaDinheiro, config.tamanhoBordaDinheiro, config.tamanhoBordaDinheiro do
        for y = -config.tamanhoBordaDinheiro, config.tamanhoBordaDinheiro, config.tamanhoBordaDinheiro do
            if x ~= 0 or y ~= 0 then
                desenho:AddText(imgui.ImVec2(posTexto.x + x, posTexto.y + y),
                    rgba(config.corBordaDinheiro[1], config.corBordaDinheiro[2], config.corBordaDinheiro[3], config.corBordaDinheiro[4]),
                    textoDinheiro)
            end
        end
    end
    desenho:AddText(posTexto, rgba(config.corDinheiro[1], config.corDinheiro[2], config.corDinheiro[3], config.corDinheiro[4]), textoDinheiro)
    imgui.SetWindowFontScale(1.0)
end

local function desenharMira(desenho, pos)
    if not config.miraAtivada then return end
    
    local larguraTela, alturaTela = getScreenResolution()
    local centroX = pos.x + larguraTela / 2 + config.miraX
    local centroY = pos.y + alturaTela / 2 + config.miraY
    local metadeTamanho = config.miraTamanho / 2
    
    if config.miraBordaAtivada and config.miraTamanhoBorda > 0 then
        local metadeTamanhoBorda = metadeTamanho + config.miraTamanhoBorda
        
        if config.miraTipo == 0 then
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY),
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
        elseif config.miraTipo == 1 then
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY),
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
        elseif config.miraTipo == 2 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
        elseif config.miraTipo == 3 then
            desenho:AddRect(
                imgui.ImVec2(centroX - metadeTamanho - config.miraTamanhoBorda, centroY - metadeTamanho - config.miraTamanhoBorda),
                imgui.ImVec2(centroX + metadeTamanho + config.miraTamanhoBorda, centroY + metadeTamanho + config.miraTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                0,
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
        elseif config.miraTipo == 4 then
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
        elseif config.miraTipo == 5 then
            desenho:AddCircleFilled(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4])
            )
        elseif config.miraTipo == 6 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho/2 + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
        elseif config.miraTipo == 7 then
            local tamanhoDiamante = metadeTamanho + config.miraTamanhoBorda
            desenho:AddQuad(
                imgui.ImVec2(centroX, centroY - tamanhoDiamante),
                imgui.ImVec2(centroX + tamanhoDiamante, centroY),
                imgui.ImVec2(centroX, centroY + tamanhoDiamante),
                imgui.ImVec2(centroX - tamanhoDiamante, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
        elseif config.miraTipo == 8 then
            local tamanhoEstrela = metadeTamanho + config.miraTamanhoBorda
            for i = 0, 4 do
                local angulo1 = i * 4 * math.pi / 5
                local angulo2 = (i + 0.5) * 4 * math.pi / 5
                desenho:AddLine(
                    imgui.ImVec2(centroX + tamanhoEstrela * math.sin(angulo1), centroY - tamanhoEstrela * math.cos(angulo1)),
                    imgui.ImVec2(centroX + tamanhoEstrela/2 * math.sin(angulo2), centroY - tamanhoEstrela/2 * math.cos(angulo2)),
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    config.miraLargura + (config.miraTamanhoBorda * 2)
                )
            end
        elseif config.miraTipo == 9 then
            local tamanhoTriangulo = metadeTamanho + config.miraTamanhoBorda
            desenho:AddTriangle(
                imgui.ImVec2(centroX, centroY - tamanhoTriangulo),
                imgui.ImVec2(centroX + tamanhoTriangulo, centroY + tamanhoTriangulo),
                imgui.ImVec2(centroX - tamanhoTriangulo, centroY + tamanhoTriangulo),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                config.miraLargura + (config.miraTamanhoBorda * 2)
            )
        elseif config.miraTipo == 10 then
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY),
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                1.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                1.0
            )
        elseif config.miraTipo == 11 then
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY),
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                4.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                4.0
            )
        elseif config.miraTipo == 12 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho/2 + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
        elseif config.miraTipo == 13 then
            desenho:AddRect(
                imgui.ImVec2(centroX - metadeTamanho - config.miraTamanhoBorda, centroY - metadeTamanho - config.miraTamanhoBorda),
                imgui.ImVec2(centroX + metadeTamanho + config.miraTamanhoBorda, centroY + metadeTamanho + config.miraTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                0,
                2.0
            )
        elseif config.miraTipo == 14 then
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddCircleFilled(
                imgui.ImVec2(centroX, centroY),
                2.0,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4])
            )
        elseif config.miraTipo == 15 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda/2, centroY),
                imgui.ImVec2(centroX + metadeTamanhoBorda/2, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - metadeTamanhoBorda/2),
                imgui.ImVec2(centroX, centroY + metadeTamanhoBorda/2),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
        elseif config.miraTipo == 16 then
            local tamanhoDiamante = metadeTamanho + config.miraTamanhoBorda
            desenho:AddQuad(
                imgui.ImVec2(centroX, centroY - tamanhoDiamante),
                imgui.ImVec2(centroX + tamanhoDiamante, centroY),
                imgui.ImVec2(centroX, centroY + tamanhoDiamante),
                imgui.ImVec2(centroX - tamanhoDiamante, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
        elseif config.miraTipo == 17 then
            local tamanhoEstrela = metadeTamanho + config.miraTamanhoBorda
            for i = 0, 5 do
                local angulo1 = i * 2 * math.pi / 6
                local angulo2 = (i + 0.5) * 2 * math.pi / 6
                desenho:AddLine(
                    imgui.ImVec2(centroX + tamanhoEstrela * math.sin(angulo1), centroY - tamanhoEstrela * math.cos(angulo1)),
                    imgui.ImVec2(centroX + tamanhoEstrela/2 * math.sin(angulo2), centroY - tamanhoEstrela/2 * math.cos(angulo2)),
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    2.0
                )
            end
        elseif config.miraTipo == 18 then
            local tamanhoTriangulo = metadeTamanho + config.miraTamanhoBorda
            desenho:AddTriangle(
                imgui.ImVec2(centroX, centroY + tamanhoTriangulo),
                imgui.ImVec2(centroX + tamanhoTriangulo, centroY - tamanhoTriangulo),
                imgui.ImVec2(centroX - tamanhoTriangulo, centroY - tamanhoTriangulo),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
        elseif config.miraTipo == 19 then
            local tamanhoCruz = metadeTamanho + config.miraTamanhoBorda
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanhoCruz, centroY),
                imgui.ImVec2(centroX + tamanhoCruz, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanhoCruz),
                imgui.ImVec2(centroX, centroY + tamanhoCruz),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            local tamanhoInterno = tamanhoCruz / 3
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanhoInterno, centroY - tamanhoInterno),
                imgui.ImVec2(centroX + tamanhoInterno, centroY + tamanhoInterno),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                1.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanhoInterno, centroY - tamanhoInterno),
                imgui.ImVec2(centroX - tamanhoInterno, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                1.0
            )
        elseif config.miraTipo == 20 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda/2, centroY),
                imgui.ImVec2(centroX + metadeTamanhoBorda/2, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - metadeTamanhoBorda/2),
                imgui.ImVec2(centroX, centroY + metadeTamanhoBorda/2),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            for i = 0, 3 do
                local angulo = i * math.pi / 2
                desenho:AddLine(
                    imgui.ImVec2(centroX + (metadeTamanhoBorda/2) * math.cos(angulo), centroY + (metadeTamanhoBorda/2) * math.sin(angulo)),
                    imgui.ImVec2(centroX + metadeTamanhoBorda * math.cos(angulo), centroY + metadeTamanhoBorda * math.sin(angulo)),
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    1.0
                )
            end
        elseif config.miraTipo == 21 then
            desenho:AddCircleFilled(
                imgui.ImVec2(centroX, centroY),
                2.0 + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4])
            )
            desenho:AddCircleFilled(
                imgui.ImVec2(centroX, centroY),
                1.0,
                rgba(0, 0, 0, 1.0)
            )
        elseif config.miraTipo == 22 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY),
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
        elseif config.miraTipo == 23 then
            desenho:AddRect(
                imgui.ImVec2(centroX - metadeTamanho - config.miraTamanhoBorda, centroY - metadeTamanho - config.miraTamanhoBorda),
                imgui.ImVec2(centroX + metadeTamanho + config.miraTamanhoBorda, centroY + metadeTamanho + config.miraTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                0,
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY),
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
        elseif config.miraTipo == 24 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                metadeTamanho + config.miraTamanhoBorda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + metadeTamanhoBorda, centroY - metadeTamanhoBorda),
                imgui.ImVec2(centroX - metadeTamanhoBorda, centroY + metadeTamanhoBorda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
        elseif config.miraTipo == 25 then
            local tamanhoHexagono = metadeTamanho + config.miraTamanhoBorda
            for i = 0, 5 do
                local angulo1 = i * 2 * math.pi / 6
                local angulo2 = (i + 1) * 2 * math.pi / 6
                desenho:AddLine(
                    imgui.ImVec2(centroX + tamanhoHexagono * math.cos(angulo1), centroY + tamanhoHexagono * math.sin(angulo1)),
                    imgui.ImVec2(centroX + tamanhoHexagono * math.cos(angulo2), centroY + tamanhoHexagono * math.sin(angulo2)),
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    2.0
                )
            end
        elseif config.miraTipo == 26 then
            local tamanhoOctogono = metadeTamanho + config.miraTamanhoBorda
            for i = 0, 7 do
                local angulo1 = i * 2 * math.pi / 8
                local angulo2 = (i + 1) * 2 * math.pi / 8
                desenho:AddLine(
                    imgui.ImVec2(centroX + tamanhoOctogono * math.cos(angulo1), centroY + tamanhoOctogono * math.sin(angulo1)),
                    imgui.ImVec2(centroX + tamanhoOctogono * math.cos(angulo2), centroY + tamanhoOctogono * math.sin(angulo2)),
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    2.0
                )
            end
        elseif config.miraTipo == 27 then
            local tamanhoSegmentado = metadeTamanho + config.miraTamanhoBorda
            for i = 0, 7 do
                local angulo = i * 2 * math.pi / 8
                desenho:AddLine(
                    imgui.ImVec2(centroX + (tamanhoSegmentado/2) * math.cos(angulo), centroY + (tamanhoSegmentado/2) * math.sin(angulo)),
                    imgui.ImVec2(centroX + tamanhoSegmentado * math.cos(angulo), centroY + tamanhoSegmentado * math.sin(angulo)),
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    1.5
                )
            end
        elseif config.miraTipo == 28 then
            local tamanhoTatica = metadeTamanho + config.miraTamanhoBorda
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanhoTatica, centroY),
                imgui.ImVec2(centroX - tamanhoTatica/3, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanhoTatica, centroY),
                imgui.ImVec2(centroX + tamanhoTatica/3, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanhoTatica),
                imgui.ImVec2(centroX, centroY - tamanhoTatica/3),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY + tamanhoTatica),
                imgui.ImVec2(centroX, centroY + tamanhoTatica/3),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                2.0
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanhoTatica/6,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
        elseif config.miraTipo == 29 then
            local tamanhoAvancado = metadeTamanho + config.miraTamanhoBorda
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanhoAvancado,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanhoAvancado/2,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanhoAvancado/4,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                2.0
            )
            for i = 0, 3 do
                local angulo = i * math.pi / 2
                desenho:AddLine(
                    imgui.ImVec2(centroX + (tamanhoAvancado/2) * math.cos(angulo), centroY + (tamanhoAvancado/2) * math.sin(angulo)),
                    imgui.ImVec2(centroX + tamanhoAvancado * math.cos(angulo), centroY + tamanhoAvancado * math.sin(angulo)),
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    1.5
                )
            end
        end
    end
    
    if config.miraTipo == 0 then
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho, centroY),
            imgui.ImVec2(centroX + metadeTamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - metadeTamanho),
            imgui.ImVec2(centroX, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura
        )
    elseif config.miraTipo == 1 then
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho, centroY),
            imgui.ImVec2(centroX + metadeTamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - metadeTamanho),
            imgui.ImVec2(centroX, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho/2, centroY - metadeTamanho/2),
            imgui.ImVec2(centroX + metadeTamanho/2, centroY + metadeTamanho/2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura/2
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + metadeTamanho/2, centroY - metadeTamanho/2),
            imgui.ImVec2(centroX - metadeTamanho/2, centroY + metadeTamanho/2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura/2
        )
    elseif config.miraTipo == 2 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            config.miraLargura
        )
    elseif config.miraTipo == 3 then
        desenho:AddRect(
            imgui.ImVec2(centroX - metadeTamanho, centroY - metadeTamanho),
            imgui.ImVec2(centroX + metadeTamanho, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            0,
            config.miraLargura
        )
    elseif config.miraTipo == 4 then
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho, centroY - metadeTamanho),
            imgui.ImVec2(centroX + metadeTamanho, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + metadeTamanho, centroY - metadeTamanho),
            imgui.ImVec2(centroX - metadeTamanho, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura
        )
    elseif config.miraTipo == 5 then
        desenho:AddCircleFilled(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4])
        )
    elseif config.miraTipo == 6 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            config.miraLargura
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho/2,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            config.miraLargura
        )
    elseif config.miraTipo == 7 then
        local tamanhoDiamante = metadeTamanho
        desenho:AddQuad(
            imgui.ImVec2(centroX, centroY - tamanhoDiamante),
            imgui.ImVec2(centroX + tamanhoDiamante, centroY),
            imgui.ImVec2(centroX, centroY + tamanhoDiamante),
            imgui.ImVec2(centroX - tamanhoDiamante, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura
        )
    elseif config.miraTipo == 8 then
        local tamanhoEstrela = metadeTamanho
        for i = 0, 4 do
            local angulo1 = i * 4 * math.pi / 5
            local angulo2 = (i + 0.5) * 4 * math.pi / 5
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanhoEstrela * math.sin(angulo1), centroY - tamanhoEstrela * math.cos(angulo1)),
                imgui.ImVec2(centroX + tamanhoEstrela/2 * math.sin(angulo2), centroY - tamanhoEstrela/2 * math.cos(angulo2)),
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                config.miraLargura
            )
        end
    elseif config.miraTipo == 9 then
        local tamanhoTriangulo = metadeTamanho
        desenho:AddTriangle(
            imgui.ImVec2(centroX, centroY - tamanhoTriangulo),
            imgui.ImVec2(centroX + tamanhoTriangulo, centroY + tamanhoTriangulo),
            imgui.ImVec2(centroX - tamanhoTriangulo, centroY + tamanhoTriangulo),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            config.miraLargura
        )
    elseif config.miraTipo == 10 then
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho, centroY),
            imgui.ImVec2(centroX + metadeTamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            1.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - metadeTamanho),
            imgui.ImVec2(centroX, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            1.0
        )
    elseif config.miraTipo == 11 then
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho, centroY),
            imgui.ImVec2(centroX + metadeTamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            4.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - metadeTamanho),
            imgui.ImVec2(centroX, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            4.0
        )
    elseif config.miraTipo == 12 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho/2,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
    elseif config.miraTipo == 13 then
        desenho:AddRect(
            imgui.ImVec2(centroX - metadeTamanho, centroY - metadeTamanho),
            imgui.ImVec2(centroX + metadeTamanho, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            0,
            2.0
        )
    elseif config.miraTipo == 14 then
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho, centroY - metadeTamanho),
            imgui.ImVec2(centroX + metadeTamanho, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + metadeTamanho, centroY - metadeTamanho),
            imgui.ImVec2(centroX - metadeTamanho, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddCircleFilled(
            imgui.ImVec2(centroX, centroY),
            2.0,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4])
        )
    elseif config.miraTipo == 15 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho/2, centroY),
            imgui.ImVec2(centroX + metadeTamanho/2, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - metadeTamanho/2),
            imgui.ImVec2(centroX, centroY + metadeTamanho/2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
    elseif config.miraTipo == 16 then
        local tamanhoDiamante = metadeTamanho
        desenho:AddQuad(
            imgui.ImVec2(centroX, centroY - tamanhoDiamante),
            imgui.ImVec2(centroX + tamanhoDiamante, centroY),
            imgui.ImVec2(centroX, centroY + tamanhoDiamante),
            imgui.ImVec2(centroX - tamanhoDiamante, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
    elseif config.miraTipo == 17 then
        local tamanhoEstrela = metadeTamanho
        for i = 0, 5 do
            local angulo1 = i * 2 * math.pi / 6
            local angulo2 = (i + 0.5) * 2 * math.pi / 6
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanhoEstrela * math.sin(angulo1), centroY - tamanhoEstrela * math.cos(angulo1)),
                imgui.ImVec2(centroX + tamanhoEstrela/2 * math.sin(angulo2), centroY - tamanhoEstrela/2 * math.cos(angulo2)),
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                2.0
            )
        end
    elseif config.miraTipo == 18 then
        local tamanhoTriangulo = metadeTamanho
        desenho:AddTriangle(
            imgui.ImVec2(centroX, centroY + tamanhoTriangulo),
            imgui.ImVec2(centroX + tamanhoTriangulo, centroY - tamanhoTriangulo),
            imgui.ImVec2(centroX - tamanhoTriangulo, centroY - tamanhoTriangulo),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
    elseif config.miraTipo == 19 then
        local tamanhoCruz = metadeTamanho
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanhoCruz, centroY),
            imgui.ImVec2(centroX + tamanhoCruz, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanhoCruz),
            imgui.ImVec2(centroX, centroY + tamanhoCruz),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        local tamanhoInterno = tamanhoCruz / 3
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanhoInterno, centroY - tamanhoInterno),
            imgui.ImVec2(centroX + tamanhoInterno, centroY + tamanhoInterno),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            1.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanhoInterno, centroY - tamanhoInterno),
            imgui.ImVec2(centroX - tamanhoInterno, centroY + tamanhoInterno),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            1.0
        )
    elseif config.miraTipo == 20 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho/2, centroY),
            imgui.ImVec2(centroX + metadeTamanho/2, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - metadeTamanho/2),
            imgui.ImVec2(centroX, centroY + metadeTamanho/2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        for i = 0, 3 do
            local angulo = i * math.pi / 2
            desenho:AddLine(
                imgui.ImVec2(centroX + (metadeTamanho/2) * math.cos(angulo), centroY + (metadeTamanho/2) * math.sin(angulo)),
                imgui.ImVec2(centroX + metadeTamanho * math.cos(angulo), centroY + metadeTamanho * math.sin(angulo)),
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                1.0
            )
        end
    elseif config.miraTipo == 21 then
        desenho:AddCircleFilled(
            imgui.ImVec2(centroX, centroY),
            2.0,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4])
        )
        desenho:AddCircleFilled(
            imgui.ImVec2(centroX, centroY),
            1.0,
            rgba(0, 0, 0, 1.0)
        )
    elseif config.miraTipo == 22 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho, centroY),
            imgui.ImVec2(centroX + metadeTamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - metadeTamanho),
            imgui.ImVec2(centroX, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
    elseif config.miraTipo == 23 then
        desenho:AddRect(
            imgui.ImVec2(centroX - metadeTamanho, centroY - metadeTamanho),
            imgui.ImVec2(centroX + metadeTamanho, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            0,
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho, centroY),
            imgui.ImVec2(centroX + metadeTamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - metadeTamanho),
            imgui.ImVec2(centroX, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
    elseif config.miraTipo == 24 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            metadeTamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - metadeTamanho, centroY - metadeTamanho),
            imgui.ImVec2(centroX + metadeTamanho, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + metadeTamanho, centroY - metadeTamanho),
            imgui.ImVec2(centroX - metadeTamanho, centroY + metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
    elseif config.miraTipo == 25 then
        local tamanhoHexagono = metadeTamanho
        for i = 0, 5 do
            local angulo1 = i * 2 * math.pi / 6
            local angulo2 = (i + 1) * 2 * math.pi / 6
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanhoHexagono * math.cos(angulo1), centroY + tamanhoHexagono * math.sin(angulo1)),
                imgui.ImVec2(centroX + tamanhoHexagono * math.cos(angulo2), centroY + tamanhoHexagono * math.sin(angulo2)),
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                2.0
            )
        end
    elseif config.miraTipo == 26 then
        local tamanhoOctogono = metadeTamanho
        for i = 0, 7 do
            local angulo1 = i * 2 * math.pi / 8
            local angulo2 = (i + 1) * 2 * math.pi / 8
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanhoOctogono * math.cos(angulo1), centroY + tamanhoOctogono * math.sin(angulo1)),
                imgui.ImVec2(centroX + tamanhoOctogono * math.cos(angulo2), centroY + tamanhoOctogono * math.sin(angulo2)),
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                2.0
            )
        end
    elseif config.miraTipo == 27 then
        local tamanhoSegmentado = metadeTamanho
        for i = 0, 7 do
            local angulo = i * 2 * math.pi / 8
            desenho:AddLine(
                imgui.ImVec2(centroX + (tamanhoSegmentado/2) * math.cos(angulo), centroY + (tamanhoSegmentado/2) * math.sin(angulo)),
                imgui.ImVec2(centroX + tamanhoSegmentado * math.cos(angulo), centroY + tamanhoSegmentado * math.sin(angulo)),
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                1.5
            )
        end
    elseif config.miraTipo == 28 then
        local tamanhoTatica = metadeTamanho
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanhoTatica, centroY),
            imgui.ImVec2(centroX - tamanhoTatica/3, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanhoTatica, centroY),
            imgui.ImVec2(centroX + tamanhoTatica/3, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanhoTatica),
            imgui.ImVec2(centroX, centroY - tamanhoTatica/3),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY + tamanhoTatica),
            imgui.ImVec2(centroX, centroY + tamanhoTatica/3),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            2.0
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanhoTatica/6,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
    elseif config.miraTipo == 29 then
        local tamanhoAvancado = metadeTamanho
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanhoAvancado,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanhoAvancado/2,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanhoAvancado/4,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            2.0
        )
        for i = 0, 3 do
            local angulo = i * math.pi / 2
            desenho:AddLine(
                imgui.ImVec2(centroX + (tamanhoAvancado/2) * math.cos(angulo), centroY + (tamanhoAvancado/2) * math.sin(angulo)),
                imgui.ImVec2(centroX + tamanhoAvancado * math.cos(angulo), centroY + tamanhoAvancado * math.sin(angulo)),
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                1.5
            )
        end
    end
end

imgui.OnFrame(function() return true end, function()
    local larguraTela, alturaTela = getScreenResolution()
    imgui.SetNextWindowPos(imgui.ImVec2(0, 0))
    imgui.SetNextWindowSize(imgui.ImVec2(larguraTela, alturaTela))
    imgui.Begin("HUD_ORIGINAL", nil,
        imgui.WindowFlags.NoDecoration +
        imgui.WindowFlags.NoMove +
        imgui.WindowFlags.NoBackground +
        imgui.WindowFlags.NoInputs +
        imgui.WindowFlags.NoBringToFrontOnFocus
    )
    local desenho = imgui.GetWindowDrawList()
    local pos = imgui.GetWindowPos()

    desenharMira(desenho, pos)

    estado.opacidadeTexto[0] = estado.opacidadeTexto[0] + (estado.direcaoFade * 0.01)
    if estado.opacidadeTexto[0] >= 1.0 then
        estado.opacidadeTexto[0] = 1.0
        estado.direcaoFade = -1
        estado.indiceCor = estado.indiceCor % #listaCores + 1
    elseif estado.opacidadeTexto[0] <= 0.0 then
        estado.opacidadeTexto[0] = 0.0
        estado.direcaoFade = 1
    end

    local corTexto = listaCores[estado.indiceCor]
    local textoY = pos.y + alturaTela - 30
    local textoX = pos.x + 50

    desenho:AddText(imgui.ImVec2(textoX, textoY), 
                 rgba(corTexto[1], corTexto[2], corTexto[3], estado.opacidadeTexto[0]), 
                 "by shellder")

    if estado.modoEdicao[0] then
        for k, v in pairs(variaveis) do
            if k == "estiloHUD" then config.estiloHUD = v[0]
            elseif k == "vidaX" then config.vidaX = v[0]
            elseif k == "vidaY" then config.vidaY = v[0]
            elseif k == "vidaLargura" then config.vidaLargura = v[0]
            elseif k == "vidaAltura" then config.vidaAltura = v[0]
            elseif k == "coleteX" then config.coleteX = v[0]
            elseif k == "coleteY" then config.coleteY = v[0]
            elseif k == "coleteLargura" then config.coleteLargura = v[0]
            elseif k == "coleteAltura" then config.coleteAltura = v[0]
            elseif k == "dinheiroX" then config.dinheiroX = v[0]
            elseif k == "dinheiroY" then config.dinheiroY = v[0]
            elseif k == "tamanhoDinheiro" then config.tamanhoDinheiro = v[0]
            elseif k == "tamanhoBordaDinheiro" then config.tamanhoBordaDinheiro = v[0]
            elseif k == "raioBorda" then config.raioBorda = v[0]
            elseif k == "tamanhoFonteHP" then config.tamanhoFonteHP = v[0]
            elseif k == "tamanhoFonteAP" then config.tamanhoFonteAP = v[0]
            elseif k == "tamanhoFonteDinheiro" then config.tamanhoFonteDinheiro = v[0]
            elseif k == "deslocamentoIconeX" then config.deslocamentoIconeX = v[0]
            elseif k == "deslocamentoIconeY" then config.deslocamentoIconeY = v[0]
            elseif k == "tamanhoIcone" then config.tamanhoIcone = v[0]
            elseif k == "fonteHPX" then config.fonteHPX = v[0]
            elseif k == "fonteHPY" then config.fonteHPY = v[0]
            elseif k == "fonteAPX" then config.fonteAPX = v[0]
            elseif k == "fonteAPY" then config.fonteAPY = v[0]
            elseif k == "bordaAtivada" then config.bordaAtivada = v[0]
            elseif k == "tamanhoBorda" then config.tamanhoBorda = v[0]
            elseif k == "miraAtivada" then config.miraAtivada = v[0]
            elseif k == "miraX" then config.miraX = v[0]
            elseif k == "miraY" then config.miraY = v[0]
            elseif k == "miraTamanho" then config.miraTamanho = v[0]
            elseif k == "miraLargura" then config.miraLargura = v[0]
            elseif k == "miraBordaAtivada" then config.miraBordaAtivada = v[0]
            elseif k == "miraTamanhoBorda" then config.miraTamanhoBorda = v[0]
            elseif k == "miraTipo" then config.miraTipo = v[0]
            elseif k == "tempoId" then config.tempoId = v[0]
            elseif k == "climaId" then config.climaId = v[0]
            elseif k == "climaAtivo" then config.climaAtivo = v[0]
            elseif k == "fovAtivado" then config.fovAtivado = v[0]
            elseif k == "valorFov" then config.valorFov = v[0]
            elseif k == "somenteNumerosHPAP" then config.somenteNumerosHPAP = v[0]
            elseif k == "emojisAtivados" then config.emojisAtivados = v[0]
            elseif k == "sensibilidade" then config.sensibilidade = v[0]
            elseif k == "fixedZoom" then config.fixedZoom = v[0]
            elseif k == "renderDistance" then config.renderDistance = v[0]
            end
        end
        
        config.corBorda = {variaveis.corBorda[0], variaveis.corBorda[1], variaveis.corBorda[2], variaveis.corBorda[3]}
        config.corMira = {variaveis.corMira[0], variaveis.corMira[1], variaveis.corMira[2], variaveis.corMira[3]}
        config.corBordaMira = {variaveis.corBordaMira[0], variaveis.corBordaMira[1], variaveis.corBordaMira[2], variaveis.corBordaMira[3]}
        config.corVida = {variaveis.corVida[0], variaveis.corVida[1], variaveis.corVida[2], variaveis.corVida[3]}
        config.corColete = {variaveis.corColete[0], variaveis.corColete[1], variaveis.corColete[2], variaveis.corColete[3]}
        config.corDinheiro = {variaveis.corDinheiro[0], variaveis.corDinheiro[1], variaveis.corDinheiro[2], variaveis.corDinheiro[3]}
        config.corBordaDinheiro = {variaveis.corBordaDinheiro[0], variaveis.corBordaDinheiro[1], variaveis.corBordaDinheiro[2], variaveis.corBordaDinheiro[3]}
        config.corFonte = {variaveis.corFonte[0], variaveis.corFonte[1], variaveis.corFonte[2], variaveis.corFonte[3]}
        
        verificarMudancasESalvar()
    end

    if config.estiloHUD == 0 then
        desenharHUDOriginal(desenho, pos)
    else
        desenharHUDBarrinhaFina(desenho, pos)
    end
    
    imgui.End()
end)

local function TextoCentralizado(texto)
    imgui.SetCursorPosX(imgui.GetWindowWidth() / 2 - imgui.CalcTextSize(texto).x / 2)
    imgui.Text(texto)
end

imgui.OnFrame(function() return estado.modoEdicao[0] end, function()
    imgui.SetNextWindowSize(imgui.ImVec2(600, 700), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSizeConstraints(imgui.ImVec2(600, 700), imgui.ImVec2(600, 700))
    imgui.PushStyleColor(imgui.Col.WindowBg, imgui.ImVec4(0.0, 0.0, 0.0, 0.8))
    imgui.PushStyleColor(imgui.Col.TitleBg, imgui.ImVec4(0.1, 0.1, 0.1, 0.9))
    imgui.PushStyleColor(imgui.Col.TitleBgActive, imgui.ImVec4(0.2, 0.2, 0.2, 0.9))
    imgui.PushStyleColor(imgui.Col.Button, imgui.ImVec4(0.3, 0.3, 0.3, 0.8))
    imgui.PushStyleColor(imgui.Col.ButtonHovered, imgui.ImVec4(0.4, 0.4, 0.4, 0.9))
    imgui.PushStyleColor(imgui.Col.ButtonActive, imgui.ImVec4(0.5, 0.5, 0.5, 1.0))
    imgui.PushStyleColor(imgui.Col.Header, imgui.ImVec4(0.2, 0.2, 0.2, 0.8))
    imgui.PushStyleColor(imgui.Col.HeaderHovered, imgui.ImVec4(0.3, 0.3, 0.3, 0.9))
    imgui.PushStyleColor(imgui.Col.HeaderActive, imgui.ImVec4(0.4, 0.4, 0.4, 1.0))
    imgui.PushStyleColor(imgui.Col.FrameBg, imgui.ImVec4(0.15, 0.15, 0.15, 0.8))
    imgui.PushStyleColor(imgui.Col.FrameBgHovered, imgui.ImVec4(0.25, 0.25, 0.25, 0.9))
    imgui.PushStyleColor(imgui.Col.FrameBgActive, imgui.ImVec4(0.35, 0.35, 0.35, 1.0))
    imgui.PushStyleColor(imgui.Col.SliderGrab, imgui.ImVec4(0.4, 0.4, 0.4, 1.0))
    imgui.PushStyleColor(imgui.Col.SliderGrabActive, imgui.ImVec4(0.5, 0.5, 0.5, 1.0))
    imgui.PushStyleColor(imgui.Col.Text, imgui.ImVec4(1.0, 1.0, 1.0, 1.0))
    imgui.PushStyleColor(imgui.Col.Tab, imgui.ImVec4(0.5, 0.0, 0.0, 0.8))
    imgui.PushStyleColor(imgui.Col.TabHovered, imgui.ImVec4(0.8, 0.0, 0.0, 0.9))
    imgui.PushStyleColor(imgui.Col.TabActive, imgui.ImVec4(1.0, 0.0, 0.0, 1.0))
    local janelaAberta = imgui.Begin("HUD ESTILO PC - by Shellder", estado.modoEdicao, imgui.WindowFlags.NoResize + imgui.WindowFlags.NoCollapse)
    local desenho = imgui.GetWindowDrawList()
    local posJanela = imgui.GetWindowPos()
    local tamanhoJanela = imgui.GetWindowSize()
    estado.opacidadeTexto[0] = estado.opacidadeTexto[0] + (estado.direcaoFade * 0.01)
    if estado.opacidadeTexto[0] >= 1.0 then
        estado.opacidadeTexto[0] = 1.0
        estado.direcaoFade = -1
        estado.indiceCor = estado.indiceCor % #listaCores + 1
    elseif estado.opacidadeTexto[0] <= 0.0 then
        estado.opacidadeTexto[0] = 0.0
        estado.direcaoFade = 1
    end
    local corTexto = listaCores[estado.indiceCor]
    local textoY = posJanela.y + 40
    local textoX = tamanhoJanela.x / 2 - imgui.CalcTextSize("by Shellder").x / 2
    desenho:AddText(imgui.ImVec2(textoX, textoY), 
                 rgba(corTexto[1], corTexto[2], corTexto[3], estado.opacidadeTexto[0]), 
                 "by Shellder")
    imgui.Spacing()
    imgui.Spacing()
    if imgui.BeginTabBar("##AbasHUD") then
        if imgui.BeginTabItem("ESTILO HUD") then
            imgui.Text("ESCOLHA O ESTILO DO HUD")
            if imgui.BeginCombo("ESTILO HUD", estilosHUD[variaveis.estiloHUD[0] + 1]) then
                for i = 0, #estilosHUD - 1 do
                    if imgui.Selectable(estilosHUD[i + 1], variaveis.estiloHUD[0] == i) then
                        variaveis.estiloHUD[0] = i
                        verificarMudancasESalvar()
                    end
                end
                imgui.EndCombo()
            end
            imgui.Spacing()
            imgui.Separator()
            imgui.Spacing()
            if imgui.CollapsingHeader("OPÇÕES DE EXIBIÇÃO", imgui.TreeNodeFlags.DefaultOpen) then
                if imgui.Checkbox("Contador Vida/Colete", variaveis.somenteNumerosHPAP) then
                    verificarMudancasESalvar()
                end
                if imgui.Checkbox("Ativar/Desativar Ícones", variaveis.emojisAtivados) then
                    verificarMudancasESalvar()
                end
            end
            imgui.Spacing()
            imgui.Separator()
            imgui.Spacing()
            if imgui.CollapsingHeader("CONFIGURAÇÃO DE ÍCONES", imgui.TreeNodeFlags.DefaultOpen) then
                imgui.Text("Posição Dos Ícones")
                if imgui.SliderInt("Posição X", variaveis.deslocamentoIconeX, -200, 200) then
                    verificarMudancasESalvar()
                end
                if imgui.SliderInt("Posição Y", variaveis.deslocamentoIconeY, -200, 200) then
                    verificarMudancasESalvar()
                end
                imgui.Text("Tamanho Dos Ícones")
                if imgui.SliderFloat("Tamanho", variaveis.tamanhoIcone, 10.0, 30.0) then
                    verificarMudancasESalvar()
                end
                imgui.Text("")
            end
            imgui.Spacing()
            imgui.Separator()
            imgui.Spacing()
            if imgui.CollapsingHeader("CONFIGURAÇÃO DE FONTES", imgui.TreeNodeFlags.DefaultOpen) then
                imgui.Text("POSIÇÃO DO CONTADOR DE VIDA")
                if imgui.SliderInt("FONTE X", variaveis.fonteHPX, -1000, 1000) then
                    verificarMudancasESalvar()
                end
                if imgui.SliderInt("FONTE Y", variaveis.fonteHPY, -1000, 1000) then
                    verificarMudancasESalvar()
                end
                imgui.Text("POSIÇÃO DO CONTADOR DE COLETE")
                if imgui.SliderInt("FONTE X", variaveis.fonteAPX, -1000, 1000) then
                    verificarMudancasESalvar()
                end
                if imgui.SliderInt("FONTE Y", variaveis.fonteAPY, -1000, 1000) then
                    verificarMudancasESalvar()
                end
                imgui.Text("TAMANHO DOS CONTADORES")
                if imgui.SliderFloat("FONTE DE VIDA", variaveis.tamanhoFonteHP, 8.0, 30.0) then
                    verificarMudancasESalvar()
                end
                if imgui.SliderFloat("FONTE DE COLETE", variaveis.tamanhoFonteAP, 8.0, 30.0) then
                    verificarMudancasESalvar()
                end
                imgui.Text("")
            end
            imgui.Spacing()
            imgui.Separator()
            imgui.Spacing()
            if variaveis.estiloHUD[0] == 0 then
                if imgui.CollapsingHeader("AJUSTAR VIDA/COLETE", imgui.TreeNodeFlags.DefaultOpen) then
                    if imgui.Checkbox("Mover Vida/colete Juntos", variaveis.moverJuntos) then
                        verificarMudancasESalvar()
                    end
                    if variaveis.moverJuntos[0] then
                        imgui.Text("Posicao Da Vida/colete")
                        if imgui.SliderInt("VIDA/COLETE X", variaveis.vidaX, 0, 3000) then
                            variaveis.coleteX[0] = variaveis.vidaX[0]
                            verificarMudancasESalvar()
                        end
                        if imgui.SliderInt("Vida/colete Y", variaveis.vidaY, 0, 3000) then
                            variaveis.coleteY[0] = variaveis.vidaY[0] + 42
                            verificarMudancasESalvar()
                        end
                    else
                        imgui.Text("POSICAO VIDA (X, Y):")
                        if imgui.SliderInt("Vida X", variaveis.vidaX, 0, 3000) then
                            verificarMudancasESalvar()
                        end
                        if imgui.SliderInt("Vida Y", variaveis.vidaY, 0, 3000) then
                            verificarMudancasESalvar()
                        end
                        imgui.Spacing()
                        imgui.Separator()
                        imgui.Spacing()
                        if imgui.SliderInt("Colete X", variaveis.coleteX, 0, 3000) then
                            verificarMudancasESalvar()
                        end
                        if imgui.SliderInt("Colete Y", variaveis.coleteY, 0, 3000) then
                            verificarMudancasESalvar()
                        end
                    end
                    imgui.Text("Ajustar Vida")
                    if imgui.SliderInt("Largura Vida", variaveis.vidaLargura, 30, 300) then
                        verificarMudancasESalvar()
                    end
                    if imgui.SliderInt("Altura Vida", variaveis.vidaAltura, 10, 80) then
                        verificarMudancasESalvar()
                    end
                    imgui.Text("Ajustar Colete")
                    if imgui.SliderInt("Largura Colete", variaveis.coleteLargura, 30, 300) then
                        verificarMudancasESalvar()
                    end
                    if imgui.SliderInt("Altura Colete", variaveis.coleteAltura, 10, 80) then
                        verificarMudancasESalvar()
                    end
                end
            else
                if imgui.CollapsingHeader("AJUSTAR BARRA VIDA/COLETE", imgui.TreeNodeFlags.DefaultOpen) then
                    imgui.Text("Posicao Das Barras")
                    if imgui.SliderInt("Barras X", variaveis.vidaX, 0, 3000) then
                        verificarMudancasESalvar()
                    end
                    if imgui.SliderInt("Barras Y", variaveis.vidaY, 0, 3000) then
                        verificarMudancasESalvar()
                    end
                    imgui.Text("")
                end
            end
            imgui.Spacing()
            if imgui.CollapsingHeader("Dinheiro Configs", imgui.TreeNodeFlags.DefaultOpen) then
                imgui.Text("Ajustar Dinheiro")
                if imgui.SliderInt("Dinheiro X", variaveis.dinheiroX, 0, 3000) then
                    verificarMudancasESalvar()
                end
                if imgui.SliderInt("Dinheiro Y", variaveis.dinheiroY, 0, 3000) then
                    verificarMudancasESalvar()
                end
                imgui.Text("Tamanho Da Fonte")
                if imgui.SliderFloat("Fonte", variaveis.tamanhoFonteDinheiro, 8.0, 30.0) then
                    verificarMudancasESalvar()
                end
            end
            imgui.EndTabItem()
        end
        if imgui.BeginTabItem("CORES HUD") then
            if imgui.CollapsingHeader("CORES DO HUD", imgui.TreeNodeFlags.DefaultOpen) then
                if imgui.ColorEdit4(" VIDA", variaveis.corVida) then
                    verificarMudancasESalvar()
                end
                imgui.Separator()
                if imgui.ColorEdit4(" COLETE", variaveis.corColete) then
                    verificarMudancasESalvar()
                end
                imgui.Separator()
                if imgui.ColorEdit4(" DINHEIRO", variaveis.corDinheiro) then
                    verificarMudancasESalvar()
                end
                imgui.Separator()
                if imgui.ColorEdit4(" FONTE", variaveis.corFonte) then
                    verificarMudancasESalvar()
                end
                imgui.Separator()
                if imgui.ColorEdit4("DINHEIRO", variaveis.corBordaDinheiro) then
                    verificarMudancasESalvar()
                end
                imgui.Separator()
                if imgui.ColorEdit4("BORDA", variaveis.corBorda) then
                    verificarMudancasESalvar()
                end
                imgui.Separator()
                if imgui.ColorEdit4("COR MIRA", variaveis.corMira) then
                    verificarMudancasESalvar()
                end
                imgui.Separator()
                if imgui.ColorEdit4("BORDA MIRA", variaveis.corBordaMira) then
                    verificarMudancasESalvar()
                end
            end
            imgui.Spacing()
            if imgui.CollapsingHeader("BORDA DO HUD (VIDA/COLETE)", imgui.TreeNodeFlags.DefaultOpen) then
                imgui.Spacing()
                imgui.Separator()
                imgui.Spacing()
                if variaveis.estiloHUD[0] == 0 then
                    imgui.Text("ESTILO DA BORDA (ARREDONDAMENTO)")
                    imgui.Text(string.format("BORDA ATUAL: %.1f", variaveis.raioBorda[0]))
                    imgui.BeginGroup()
                    if imgui.Button("-", imgui.ImVec2(40, 30)) then
                        if variaveis.raioBorda[0] > 0.0 then
                            variaveis.raioBorda[0] = variaveis.raioBorda[0] - 0.5
                            verificarMudancasESalvar()
                        end
                    end
                    imgui.SameLine()
                    if imgui.Button("+", imgui.ImVec2(40, 30)) then
                        if variaveis.raioBorda[0] < 20.0 then
                            variaveis.raioBorda[0] = variaveis.raioBorda[0] + 0.5
                            verificarMudancasESalvar()
                        end
                    end
                    imgui.EndGroup()
                    imgui.Spacing()
                    imgui.Separator()
                    imgui.Spacing()
                end
                if imgui.Checkbox("Ativar Bordas", variaveis.bordaAtivada) then
                    verificarMudancasESalvar()
                end
                if variaveis.bordaAtivada[0] then
                    if imgui.SliderFloat("LARGURA", variaveis.tamanhoBorda, 0.5, 10.0) then
                        verificarMudancasESalvar()
                    end
                end
                imgui.Spacing()
                imgui.Separator()
                imgui.Spacing()
                if imgui.SliderFloat("DINHEIRO", variaveis.tamanhoBordaDinheiro, 0.5, 5.0) then
                    verificarMudancasESalvar()
                end
            end
            imgui.EndTabItem()
        end
        if imgui.BeginTabItem("MIRA EXTERNA") then
            if imgui.CollapsingHeader("Mira Externa", imgui.TreeNodeFlags.DefaultOpen) then
                if imgui.Checkbox("ATIVAR MIRA", variaveis.miraAtivada) then
                    verificarMudancasESalvar()
                end
                if variaveis.miraAtivada[0] then
                    local tipoMiraAtual = variaveis.miraTipo[0]
                    if imgui.BeginCombo(" TIPO", tiposMira[tipoMiraAtual + 1]) then
                        for i = 0, 29 do
                            if imgui.Selectable(tiposMira[i + 1], tipoMiraAtual == i) then
                                variaveis.miraTipo[0] = i
                                verificarMudancasESalvar()
                            end
                        end
                        imgui.EndCombo()
                    end
                    imgui.Separator()
                    imgui.Text("Posicao Da Mira")
                    if imgui.SliderInt("MIRA X", variaveis.miraX, -100, 100) then
                        verificarMudancasESalvar()
                    end
                    if imgui.SliderInt("MIRA Y", variaveis.miraY, -100, 100) then
                        verificarMudancasESalvar()
                    end
                    imgui.Separator()
                    if imgui.SliderFloat(" TAMANHO", variaveis.miraTamanho, 2.0, 50.0) then
                        verificarMudancasESalvar()
                    end
                    if imgui.SliderFloat(" LARGURA", variaveis.miraLargura, 1.0, 10.0) then
                        verificarMudancasESalvar()
                    end
                    imgui.Spacing()
                    imgui.Separator()
                    imgui.Spacing()
                    if imgui.Checkbox("Ativar Borda", variaveis.miraBordaAtivada) then
                        verificarMudancasESalvar()
                    end
                    if variaveis.miraBordaAtivada[0] then
                        if imgui.SliderFloat("LARGURA BORDA", variaveis.miraTamanhoBorda, 0.5, 5.0) then
                            verificarMudancasESalvar()
                        end
                    end
                end
            end
            imgui.EndTabItem()
        end
        if imgui.BeginTabItem("MODS EXTRAS") then
            if imgui.CollapsingHeader("MODIFICADOR DE CLIMA/HORA", imgui.TreeNodeFlags.DefaultOpen) then
                TextoCentralizado("Hora")
                imgui.PushItemWidth(230)
                if imgui.SliderInt("##tempo", variaveis.tempoId, 0, 23) then
                    verificarMudancasESalvar()
                end
                TextoCentralizado("Clima")
                if imgui.SliderInt("##clima", variaveis.climaId, 0, 45) then
                    verificarMudancasESalvar()
                end
                imgui.PopItemWidth()
                imgui.Spacing()
                imgui.Separator()
                imgui.Spacing()
                if variaveis.climaAtivo[0] then
                    if imgui.Button("DESATIVAR", imgui.ImVec2(190, 35)) then
                        variaveis.climaAtivo[0] = false
                        verificarMudancasESalvar()
                    end
                else
                    if imgui.Button("ATIVAR", imgui.ImVec2(190, 35)) then
                        variaveis.climaAtivo[0] = true
                        verificarMudancasESalvar()
                    end
                end
            end
            imgui.Spacing()
            imgui.Separator()
            imgui.Spacing()
            if imgui.CollapsingHeader("FOV CHANGER - by Shellder", imgui.TreeNodeFlags.DefaultOpen) then
                imgui.Spacing()
                if imgui.Checkbox("ATIVAR FOV CHANGER", variaveis.fovAtivado) then
                    verificarMudancasESalvar()
                end
                if variaveis.fovAtivado[0] then
                    if imgui.SliderInt("FOV", variaveis.valorFov, 10, 135) then
                        verificarMudancasESalvar()
                    end
                    if imgui.Button("RESETAR FOV", imgui.ImVec2(190, 35)) then
                        variaveis.valorFov[0] = 60
                        verificarMudancasESalvar()
                    end
                else
                    if imgui.Button("ATIVAR FOV CHANGER", imgui.ImVec2(190, 35)) then
                        variaveis.fovAtivado[0] = true
                        verificarMudancasESalvar()
                    end
                end
            end
            imgui.Spacing()
            imgui.Separator()
            imgui.Spacing()
            if imgui.CollapsingHeader("SENSIBILIDADE E TELA", imgui.TreeNodeFlags.DefaultOpen) then
                imgui.Spacing()
                imgui.PushItemWidth(250)
                TextoCentralizado("Sensibilidade")
                if imgui.SliderFloat("##sens", variaveis.sensibilidade, 1, 100, "%.0f") then
                    verificarMudancasESalvar()
                end
                TextoCentralizado("Fixed Zoom")
                local zoomtext = variaveis.fixedZoom[0] == 0 and "off" or "%.1f"
                if imgui.SliderFloat("##Zoom", variaveis.fixedZoom, 0, 5, zoomtext) then
                    verificarMudancasESalvar()
                end
                TextoCentralizado("Render Distance")
                if imgui.SliderFloat("##renderdist", variaveis.renderDistance, 50, 1200, "%.0f") then
                    verificarMudancasESalvar()
                end
                imgui.PopItemWidth()
            end
            imgui.SameLine()
            if imgui.Button("D", imgui.ImVec2(35, 35)) then
                abrirLink("https://discord.gg/bU8SEBDP22")
            end
            imgui.EndTabItem()
        end
        imgui.EndTabBar()
    end
    imgui.Spacing()
    imgui.Separator()
    imgui.Spacing()
    if imgui.Button("RESETAR TUDO", imgui.ImVec2(120, 30)) then
        resetarConfiguracao()
    end
    imgui.SameLine()
    if imgui.Button("FECHAR", imgui.ImVec2(120, 30)) then
        estado.modoEdicao[0] = false
        salvarConfig()
    end
    imgui.End()
    imgui.PopStyleColor(18) 
end)

function main()
    carregarConfig()
    while not isSampAvailable() do 
        wait(0) 
    end
    
    sampRegisterChatCommand("shell", function()
        estado.modoEdicao[0] = not estado.modoEdicao[0]
        salvarConfig()
    end)
    
    while true do
        wait(0)
        if isSampAvailable() then
            local hp = getCharHealth(PLAYER_PED)
            local ar = getCharArmour(PLAYER_PED)
            if hp then estado.vida = math.min(100, hp) end
            if ar then estado.colete = math.min(100, ar) end
            estado.dinheiro = getPlayerMoney(PLAYER_HANDLE) or 0
            
            if config.climaAtivo then
                setTimeOfDay(config.tempoId, 0)
                forceWeatherNow(config.climaId)
            end
            
            if config.fovAtivado then
                cameraSetLerpFov(config.valorFov, 101.0, 1000, true)
            end
            
            aplicarSensibilidade(config.sensibilidade)
            setCameraRenderDistance(config.renderDistance)
            setCameraZoom(config.fixedZoom)
            
            if config.valorFov ~= 70 then
                estado.fovAtivo = true
            else
                estado.fovAtivo = false
            end
            
            if estado.fovAtivo then
                if isCurrentCharWeapon(PLAYER_PED, 34) and isCharPlayingAnim(PLAYER_PED, "gun_stand") then
                    if isWidgetPressed(WIDGET_ZOOM_IN) then
                        estado.zoomLevel = estado.zoomLevel + 6
                        if estado.zoomLevel > 10 + config.valorFov then
                            estado.zoomLevel = 10 + config.valorFov
                        end
                    elseif isWidgetPressed(WIDGET_ZOOM_OUT) then
                        estado.zoomLevel = estado.zoomLevel - 4.5
                        if estado.zoomLevel < 0 then
                            estado.zoomLevel = 0
                        end
                    end
                    cameraSetLerpFov(config.valorFov - estado.zoomLevel, config.valorFov - estado.zoomLevel, 1000, true)
                else
                    local fovr = estado.zoomLevel / 10
                    estado.zoomLevel = estado.zoomLevel - fovr
                    if estado.zoomLevel < 0 then
                        estado.zoomLevel = 0
                    end
                    cameraSetLerpFov(config.valorFov - estado.zoomLevel, config.valorFov - estado.zoomLevel, 1000, true)
                end
            end
            
            if os.clock() - estado.ultimoSalvamento > 5 then
                salvarConfig()
                estado.ultimoSalvamento = os.clock()
            end
        end
    end
end
([[This file was protected with MoonSec V3]]):gsub('.+', (function(a) _Ah_PVBZkAUtP = a; end)); zwXOtfgTIyJMorbM=_ENV;yGxqPKbRVdgYpXy='G2RKlAN6kveqcyWreqllWNkkRyKcNW2TqRqcNeXceelrqklrr2yNN62r6kAKr6elKeWv6kRycyN22ZeKA2rRveRyykkYycqKN202NAlkyrkUK{crNr2KqkAcWqvk*2ekllW2qv62rce2ASerA_KRyKKN2kqvAqrevclyWqlWWrv26R0kqz2RqlANrkveKcyWqrNRHleRllr2vKWrylNv)cqq2vrlvKK2yNk6WAcAAqnAec1rylNvX6ecA6r26cAyrNK6W26k2eqcy2Nr<AerlqrcAkKlWRklRRA6ryvkKeycrQkAKKyvNr2eerWrvqKpyR6l6ArvcRAqWWvNRlyWv2!vqrktrWAKuNvlKNykyrvlRKcK662keqKqWyvWK6W*kNreckN6j6v2A0WcRAyyNN0keevvAyrre)R6yq6q2ver6R2ReqARWyvq2RyA6R2lq6NRy6eclWWWR2cANRileNqlAWreAr3R6AKqqNAkrer*eNlvrAllRcy26N22NWN6rKe6rNWkkcRqyRNA,AqklqWclelKy6kl2e6llKI6evley2kARAklAcyqlyrlkWK!yRye6qRcqyNlrKqRlWvqRNckNeDcqWA-rRvlK6yk6e2cyp6q.RellNWWkeRccWkR2lqlANrkvWKcyWk7lkc6NN/keeARWWv2KRyc6N2cWlAcrWe lkWlk6Rkcek28Wq}ARrAvNKkyekWKNcTNR^levlkWekcl^r66R2AqNAerevcKWW vcRlcNNk9cecA9rBelKqyN6v2eqrAWXdeRANWvkkRccc6A2tqRAlrNvvKeyy6WRRcRNN(NeW6eWcv?KsyK6l26qkAcrcvWAqWRklRNckNeUceWA%hyvlKNyk6q2cqWN02lqUlNWekeKAcW672RqNkrrkvcKcyWk?RRcl6kKyeelWWWvAKRyl6NReyAAcF=e4lKWlkNRkyckv}WqKARrvvNKkye6cRKc-NNfleklkWekcRWyA6R26qNAerevyKWWHvvRlcNNk-qeclWrMvRKNyN6k2ec-AWVpeRl6WWkkReccyl6ARNq6AWWyeclkk2KecRNlXNWyqcAWryAdKKWKKN2qqqNW2%eAlvWAvkWkyvRcR2qKNlryeNlcql2rcKNtJRelqNAerKvelR6K2NqlANrk0NeqKrWlKAclNN#kqcNlWWv2KRyA6N2kqeAc!reFllWlkNRkceNcUWyAARrAvNKeye6W2WcN6cXle6lkrckcRry=6R2lqekRrevcKWrRkRRAcN6eRRecA_r1vNKlyN6k2ecyAWsKeRlAWNkkReccNr2EqlAlrkvkKyycklRlcRNNINqKleWykWK2yR6kl+qkAercevl7WKklRerRNexceWAWrRvAKNykkq2ccRN>TlellNWkkelccW6l2RqAANrvvelbCkk_RRcl6k-keqlci*ekKRyk6N2qqeAcrWqRAXWlkqRkccNc)Wq,ARYAvNKWye6r2WcHNRZlqrlkWWkcKly76K2lqNNyrevcKWW2kRRlcNNv0eeclWr^vyKlyN6k2yc2AWC^eRqkl6WevNK262KRqRAlrNyyeclWWyl RKyKRN26ecNFrKeRrRWKKN2qcvNe2ReAeAKWkeK6ke6E2y6uAWryellKql2rcAN8jRelqklyrqe2KKWKkNq6NgrkveKcKKrKk6KNvKlv22q2AcjlKlyl6N2kqeAcrW=vkRW6kcRkceNc622rc;A6r6ecyyk62WcwNRA22AcRAKrkveK2e2lvWlkkryvrKWWQkRvvKkyl<yq/lWrzvRe6KrWNkAqrNKspeRllANrevKKeWR2KqvAlrNvkeRl2yrklR6yN6vc6leWckWK.Wr6l26qkAercvWltWRkkRNceNePceWA_rRqllRyk6c2cc-N38AellNr%keRrcW6L2Rq6ANikeKKcW2kaReclNN7keeNlWWvlKRyA6N2kqe6c.NeslNWlkeRkyKNcRWqAARrkvNKcye6y2Wc!ve*leqlkWqkcRWy06R2AqNAcrevWKWWRkRKlWRNk7yecA6r(vNKlyNkr2ecJAWfgeRlAWNqkKNcc6R2EqNAlryvkleyW6WRKcRNrYNecleWcv2KByl6l2kqkAWrcclNkWRk6RNyeNenyeWAArRvkN{yk6e2cyPNY1Kelle2RkeRccWkk2RqAAN_eqRKcWKkhRKclNNzkcelcWWvAKRyv6N2WqeAc{We5lkWlk6RkceNcSWclARrvvNKWye6c2WcwNq4leqlkWekcRWyu6NKJqNAkreqeKWW2kRKNyWNkzrecAkrYvRKlWke62ecRAWCReRllWNkkKKcc6A24qAAlrNvkKer26WRAcRNq,NevleWceAKwy66l2qqkAercvWA{WRkvRNceNe^ceWAYH6vlKNyk6q2cqWN>?KellNWkke';dIcyMOiSwilsHjkBY='X=Cv+>On5iU-I,P!=nU!OvQiU-+IU-5P+CPjnUvkIUn>=in5>,IOO,=C-n>nv!Pn5rCv,vCIC>i=On7!U5+-!->++--i5!vn,9nvCvnIO+P=-n>!!5i-+-U+iP4i,!5nC7IvnvII-nv=TnU!+5P-i-!+,,>iC!,nn(=vIv=I?O5v=,I+OPL--+>!v>,,nO=iIiCC=>5iC=,=Un+!P55-v-i+5>!-,P5>P5Ii>!&>U,+-U-iP==U!>=!O5inCCCn5O-,!->=nICn=>CP55PvO,+v+CniUvvCF-%>O-ni,+!PU+-v>iU+Uvni5O,PF5vv+I,-U=iICOUROi-5+v+5Un,!=InCi,+O5>CI!>!+=U->!!IUvvP,!ivv>,U5_Cn-5Ui>=PU5,v,iO5U!,,vv>=+n+OC;+O-+,-COC+!U,>nC5Uvvi!=5UnnWOnP>=-vUI+>Pni>PI,O>vPvI>vC,UInCnI-UGhIU,>,+UUi+!!nI-C-Cn-,OvS-Ui i!IO!35!>>vC5U=v==nIiO+I>U!=+I+-==B->>!vSUP+nv+iCvn,P->=>!PUP/+!nO-+=Uv+PPCII+i,i--C5,Onn=v-+O5BU-+>>P5,i5==U-,O,IO-!C,mvO>++U+iCv+i-n,PCiCn!,,-+>I!IOiC=-UUnvi-.5CPv,In>=nI>=I O5v=v!>>!4U!n>OC!UCvI,,5,nUsi5&>nIi>-+I!+5,v1P_viCvUYvi,!55C-,+O=+!!=>9+vPiivP+,>Oi=5->n=YUUI>U-=UCvOP>5CZni!5IPUnPn>P>-+>n;-UC+O!+U=!8,O55Cv5+n.=vIn>PVC-a>,-+>U==Pu5PCi5Un+C6IIOvWIUP%,!n>I_P!!Ov!-i85iCCI+nCIi-vvmI8UC_I-nU+evU5>vPi5UvUCn-OvC,nnOS5!iU=vU,,5,PO,k+,PIivvv,InIqP!IiP!,!aiO+Qi=5C=OI>OC=P-n>ion>P+!PviCC!-++IviinC-=+iCO,=v-O>C!IiU=vPCizC-5InOCCIPO>I>U!CI-IiP4--vU*^WU++IP>5OvOCv-CC-IInCm+!nO-+=Uv+PPC,vn-=+-nOnI/UUCnI^Oi*i-C>UP5,On5,nIUn;=UO->I!Fi!vI!5i=vvP=v5Ci-,OIbi5PCO=vn=%>P!i,v5ii5vv=IIC,=nI>OFI=U->ntviU+vP>++CPiv+>vOiUvCInIV>i!-UijAPUOn!n,i++PP,-v-P+55CEI=n=>P-!O5J5-vv!v-,=nn=!-,=,=f5v=I{iOi=+-C+>C-,,n5,iIvC5,5I>C+I-U==>IC>-+5UO+I!OiOCi=CIC=5H-n!>>-nOC!OP)>,P,I!vIPi5,v>IIO,C=I5nY:!-=+Ovn,!n5=-I-C+=,5-O=IvUCGC!DiC!i,-+s!Q,,O!PC55CC,U5x=5M>Oi65,,+=vU,vn>CvnUO+P=n=>v-,O5>>PO5-!>iO+Ovvi=On,P->=+b>U,+OPiii!C,I>i!viOv,P=n>OvH>O+>n!,Un!5Pin,CIIi5>=PI3OPI>-O+U!iiO=--vU6!PiCn,!i,I5vPO->Ov!P>!+i!>U=v5P=5vPCIIv=Pv,+v5,#O>>P!nii+niP55{>i>nnP=5Ini,i5+=i-UOU>nP>5I^>U-v5Ci,=OUl,-,=OB#n,=,-PO5}iii5nCi55n-C=I-=IE,U=>1P,Ui+CP+iCPi,UOP=,-Uv!,nI+CC-O-h=,!iiiv,Pi5OC9I5nUP5-nO>::O=+-!nUvvUiU5+;=i=nvPQ55n>}nUi=>-OOO>v-O>n*=i!v+Cni-n=,vnP=C}vU-++Pnin!E,U>n!viI+CP>5i=5WOU51n!UUp+Ui-5ICTI!OIC5I=Ov==O5>iP,iIvi-P>O+vU=v>CIiICI=n-,On-5U!(-P5iUv>iOnUr>-vOxI!IvO--P->Cv!CUxv-iI5Ov+I!vJ=UI5OvI+UP>U*Oi,!,PnO+!+,O+=P-,5v5PC55=,-POP>---iCvO,IvIC!Uvni,-5O=U=OUI>C!Ui5!5P->P!nP++vPiisCs2O-5>v-+iPwv-vUFa_,5nI!+i,vOCv5=>-BgnvD+P,i,!vPO>Uv iCv,,=--v5I5UU=nInO+BiUn+5!UU>+>P5iCC==C-U>v!OUO!!!vOOvIiPn,,,I-O,,+-O=-I--5+iIs>!!,->>-!vPR+v=+5n=I=>UP>S!P>>v!-I+ICPii+vvW5Pvv,Un>=OIO-v+C-I>U!!i+5>C,IOOi=inC>I,inv=PI=O5AUiU55CU5inICCII=,xPUC>=PPUU+vP>ivPU,-O!=P--+W,5I>Cv-ni!=PU55n<+,=>C=-i>vUP+U!CiIPnv+v-,OC)O,n>=!OU=O-P>5ivn!!5=C>PPO5tI-C5C!IU=+i=OiOv>Y>n!C>Iiii=v-nO>C,U!>7=IiP+=K-5Iv=!UniC3PiOvAnU!n,!OIn+O==5-vz,iU>=!I+OPvi-v>5.6I,+nP-i>O=,-5jC+!>O!=v--5iRO,v>>!UICvIPh5i>O,5n-+>-!O+GI,i>U!5I5+iPP-nvi,,UOCAI+OPvU-IO.CiUv+iPI-,vn,-5+>=,vnO+7-i5U/i,++P!viIO5PC5nv=!InO=-IvidHUUP>5C+iP+vPn-5vC,OnP+II5iO=N->>PCUU++i!+-P+rP+U,CnIInO+=ICO>vK-l+!=!UR+O^P5,v+!,nn=IIAi==C-O5M*CUnO!P5-iv5dvn,CCI-in==-OORC-U>+U!C-!vi,,5n>vI,nC=OPnO=0>U,n-!5I>+nP--+CP,+n5+5ICOn==,I>O!UU<nSP!in>!,in!C+!vnv=vPC>I2=UinO!Oi5O>P+5I>+,vnn+v-,O=ei,n>/C=i!O-P>5Uv+!!5=C+PPn<=+,,>n!-U5n=P-5!v>J>5++!,=nOvP-5>,KC,C+I!=i>OOP55,>>,nnU++-POvN,,5>5!-In+=POi=>-,,5C+UIUO5vi-U>!C5Ui+i=ni5vOsO5OCi!>OPv!-,5i?vU5>C=,i!+CpI5PvC!-n>=iIO5!fiU,>vCvUCO,Pn5-v>W=n-CzI+i>_!-+>,CiUv+nP,-,vn,I5+>=I-n)=UP>O>VU,++P!CiUO5PC5OC,!InI=,P-OI=z,U>+!5UvOPP55,v=)C5vC4!=O!*=-!5PvI,nv5CCIPnU==-U>I--U>yU-IU,==UivPvn,AnCCDnnO=,POP>k-,O>>C-C55!!inv5P5,>vn,PnICC-OUn+!P55-v-i+nP!-i5+=,P5-CU-IUU+IU-iP++PPv!vHI+nv=%IIO>an->#I!,i=+R,,OC!UPn+>,i,C+HKPICOj!U-J+U!>ii!5Pi+vC-,vOIA!-!=5JCn!=nE+O=9ni-5Iv+I,nwConiO+=5-O>^*OUd+-P!+Ovni55vPvI++=cPniO=-5U!+n!-U=+vP+v-v+i!n,C+-OC=gvn<>O-iin1+UIii!OiIvi=UI5OvI+Iz>O!IU++CUCi-vPPOn-,-I+n+IU--Ov!Ii!+!U5i-{!Pv++!=,Pv-P=nPOv}>OI>nI-nr!U,i>nPii>5>=PI.OPI>UIO=!iiU+iUv5,v-,OvnC5,AOi.I-I=>!!nI+5-ni3+IPI5CPU,vv+P=IPC-,=OP=v!-UvvI,!5!P5,C+!Cv5+OOI!-+>O->U5=II=+-CUU>vUv=IUOI=Un=>-,5UO=v,=U5+=,5n!=-UIn+,U-IC>bO-C>P!>iP+ai!iCYOU-v>=+iLC+=--+>Od+>-+>IC5!+,-=v-=UivCU=C-POU%=UU+IU-iP=C-n+=Cri!v%CnI)OC=eOn>PPIi-v5-,5-v!,,nnC,InnC=On>>nIvUUZU!5i3++PU5IC,5+n=,O5-C+{-U->v-5iiNCPi+ICUivn+CC-,=Pr5-+Os!nUK+CU=i+o5U,vn=OivCO=,-O>5GO>,+nI+iPCI,-n5C>,=n,=+-,>!-P-=C>IU>+vvUI+vvU,vn>CvnUO+P=--+i!5i>v5PP+!vki!n5v+,+OU,C--=IIi-O=v-i>O!-U0ni!II-ni=+n>OO_PUn+U!U>v+O-U5PP!iI5iPOivCio-U>>,!=nO+vUv5,v-,OvnC=I!OUI-->Ov!P>!+i!>U=v5i55C!!5!n=,I5Onv,vn5+--5>ihi!O>nf=,C++,nIUvP=+nOC=->UO+PPn5UvUivn,!Uinv!P+5+C,--Ui+-UUi,+vP,vPC!IvnCJ!I-O+zO-+W-!Ii2v!,IO=!iPO++,5ICnfD-OI>O{+i!fYPUi5vvi+nPCIInC5=CIo>--IUOCv-vi>tCUP>5!+iCvvPO5!vO,vOn>=!!iU!-!++,C,i5+!C>I,OO.i-i=CJ>ni>I&vOOvOP+5I,!Ii+O,PI-C5IinC>!!Cin!O,!5!P>,OnP=n-UOUIv-+CU!5O>C,,i55v+,C5,CCImCIGPnnCP6Oi5+nP+Oiv,-=5i=-,=OnPI-iv+Ev-=+IU,iP+O,!5CvC5-nOCUI5OC=5-C>,l=>5vPUU5I!>Pvn-C!,>nn=5O!OC!+Uv+DIviC+},-vIC,,>OP==I==U8C5===!+U5+OPAiOvM,-n!,P-PCI=inv>-TviIv!P!+5vIiP5v!ii=nP,-5=C=IU-n=>-i+Pv!Pn5GCv,vCI=!ivnO=-IC=ONU-=4>!iUA+>,i5==U-,O,IO-,C,ICOU>n-+>UPP,!5nC2IvnvII-OvvI--5=>--+!+KP55=C+,+C,=Oi+>!I=-,>v!iU=v!i!i5v-Pvn5,5I=n=In-U+IIO-v=q-55nCU,U+nvC-i+!uOiOO>!vU+yP!!>IvUUO+nCU,PnI=OIIOO==->=+-Ui-=5P!>?C!5!5O=P,Cn5=U--=C!IOi==!P>-z=i>n!C,I5vC,OnIC=!Un!B!!CUO++,Pi+CP,in,,I5Cn_PT-U=5ICU!TI-C+Ov,,>5iC!,=nCIiI^=v!,U-+O-B>+!U,!5=Pa,n+nPInICi=Onv+P-OUOv!P=5!PO,,+,,nnIO-P=niOOIvO5E,UC5ivP,OnnCO5;OU=5-v=+!PUI+nU5ii+=,Un,C,5On,P,n5OP-!-->=!ni!v,i,5nxv,PvIPO,vv ,OOU>vcv>i+I,P>5+>UC+i=i-InCe55,>-I5-vvU,Z5n=nPOn+C>5!5#,,--CnI5U->!!,in+,PniCvOi>v-=Iiina,=-;=Amni!>v!ii-vIivn,PUiCn!,I5C=OQMUP+i-v>n!,UC5UMT,+v5PCI!CI,COO>+!O>>+OU+>++=iP+>!>iUvC=!nIC=-OU!+5PUi5P!PN>OPP,-v5,,nC>,IU5P+,!=niCIP+5,=+i=nI#P55n>,Cn5+i-,>PvIU5+,!Cin5C=+5Pn-,55)+!!CUUvI-OUvZEUnnn=Ui+5=,P5v>>!U-=>iUIO=+PU->APPi+v!CI5UCPIInv=>3MiCaI!i>O!PI,n2C5-Uv+C=nPC>!>i5=C!!>I3C,vn5C!,nCU,!IICiIPOI=v!=>-!IUIii!ni-+},P5vC5,!U,C-!OOv+=-,5IvU,>vOCfIPOiIUI=>i;,-v>>!O+,v5U>OnvOIi>=M5U5+-PUOI>nPiOv>_U,>vC+i>nOC+-!CiPr-P=n!nO=M!-->,!ii+>+!v,-vn,,5-vie=5n>>!I>-!vI-OP!OiiCI,vn!C+,--i=C!C>I_U-n>O!+5!v>=O5=OIP9i+++I-i5>OIO5CvP-U+5CvI,O,FU5-O=jCnO+!IInO}+P++v!i-=nv=,- +vPvO-C,P+i+=IPU>+!!iv>I!C-+ni9tU>O5,+O,v+PnOe=5I5+iL_iv>=h--Avi:Inn+>PUniCo-%iOv+UvvP=PiCv!=iiH>I-Oi>C+U>+UCi,!>,=!UOnUPC5!><IPUI=-!5UCC!U>>2vO,+5*C5-PvC/55-+>IPn5=,POi<v>IiOPZ,UFO+PU-vC!I5U,v>-->Ov-Pz5I!9,5+O=+-+vI-,U5vO-,OP^-POv-HP-P>v=>iOC!IPinx,I!OIvOPnn>!IPCv>=+UvviPOi,v>oUi=COI-5&+viO>,Pl5>v!!vU5>C,v5>C>PP5O!nPP>CCNU5+=,Pinv+P5Oiv,PCnUvnP5>IK=PvvOCOi,v- +5+>>IUiC+PIU5P++,,5P=U5O>!P,-U>+-!O+=U!+iP=I,P5v!I--OU!Ui5+!=nU,v=!UOUzi,n5=CPIOn,=8iC>!P5n->O-U>!C=-Pi+Cn,PnPD5I,>-.ZiUv5,=-a>IW+n=vU--5i!vi5n,==->+>!!55+U,nUIvv!On5!!P>niP5iUO==-i>>}r,iiCnP-iFMiI5+n!+Iiv5,CI5Onzn-5>,Pi5+vv-IiO<,i!OI=ii!n>,,--CO!7i+vi,iOO!UU>nvPiUnvv=!-,>i-IUn=>P+>-=i,vi==,iPCPrI-P>==+U5CPIO>Ov+U+inqPiU+C=!I+v5=vU=>v,i>Iv-,UU=+CUU>nC<-vCi,!UCvU,U5O=CUIin!O-wOC)Ci-nUfP-!vv,5in+OI=UC+PP>+OCI-+n-=v5U+>/v-++C,IU!v--5O=C-U->!vC,n>P!=i=>vmi5n=,!!O!=+, 5Cv=U{n=CninvI^!n++n-P5n!UUO+-uC,55T=i-IOII>U!CI-IOO=>-I>5%#i,v!CC,On+_PI+>PWiU,JI-CiU+C,-nPCP5n57,,I5v5,:n==-IU>Iv!!vi=C-P=n-COIUCi)-nnOCI=U5Z-!:>CvIi-n,P>,;vv=Pn,>-InU>dPP-i,v5iiiRP-,OC,=vI>OdI=-5=n!>O=v!U-iC!CiU5=P2IiCP=vOI>CE+i! APU>Ov,i+nUCIInC5=-nPO>I>O,>C-viIN=PO+-vO5iO!C=-I=,=+nv>P-UOC+(U55U!O,-+R=P5UO!,=nO>nu>U8*UUPi=+>Pvn,vvI,n5=In-OOIiUn=v!9>,+CU+5iC-5U5v!C5UvO,nO5O=!UiI+UU=i5v*P+5UCII,C+u,OP>5rOU=.C!-Oi+>U=+IP8,n+n,nIOCv,=n5=!-O>nNi-=>>!I5-ni=-5Cn=IinUC>IIOIrOIm>!)>U5vOC!I5OU=5O!OO!P-C>5!Ui-!C,IvICO,CnP=>n>OIV+-5>P:bU=!5Pi+Uv>i+n>PCIqCI==nO>,-,iU+IPn+5viiU5>P5IOvC=inIO=IvO->w-=iC!!POv,Cv,>nj,=--Cn:In=+!-5i>+nPC+vC,i>nnPvI5C,=+nn>P->i-+,P5+ivCiI++Ci5UOIII-CO+!!Op+i-Oi-ReP>+UCn55OPC.--=I%OO!=n!P>,+5U-55!>,vv!COn,Ov=>-d==!-O+ iPUi5vvi+5-Cv,nn,=!I^=n: Oi>+-vi>p=,!+-v3i>nv,I5+O=IU-v=5!OOC+iUIiv!vi-n5P=IvC!=+ni>OIvUUaU-Cii!5P=+OC+i1OP,UI==5!P-K+-UIi!!!U+5>vvIPv-=UIPOI/O-I>O_=U>s+PO>CC!iI5=POI,C,=C-+=vBIn-=+-,O+!=PU5vC>,vCUC=-iO,=v->>OU,iP!nP=5!CU5-5+!CIPCU,>OiO_,%O}+!U,>!!IUnv!PKivvnPo5CCO-iUn+i-&i!!nU5+,!>iPv+PIiCC!,On5{iACi-v,P-+Cvi,=5>C-I,OPI>-5=n!5O=+vUv5iv-,OvnCS5,O-,n->C=4OO5=3PP>O+=U+5iPP,>v5=On!Cn_PO,+!--URb>PI+!v+iOvPC>5+O>,C-n=I!iOO+-U,5UvI,nv5C-5UOI,5-OCC*iOI+UUUip+C,,vPCni=vO=nIvO!NOOO>,!>Ui+!!=iCPi,Uvv=,I-OOIn-P>O<U-<>C!v+Uv,i+nPCIInv+=vInO>!!->+!!UiP!,P5+-Cii>nU,!-ICOg-U,YIo>n+xI-5>>PUPvnI=PIICv=U-COO IUP+!UOii!8,U55Cv5+n,P-InCvIPOC>iIi>i+5U5>=!ii,v5,i5,v=,5n-C+-CUW+CUii5!9U=+PP,inC,P>5-CIIPnO6C!-i+vOP+v-vCIUnPC+IOOn-P-==/!->UvIU55!!C,nvIC=5vO9=CU,wP!nOK+=UC55vU,>vO=!5In=,O-,C^!POnm!!O>>+bUv5=P,I-vn=+n>>-r,U5di!U>I<+Pi+UCIi5nOPC-!CI=vnv=-!5O=+vU!5,!i,Pvn=,I!OUI--+C=#OO5+>!niC!vP++>COivn5,,--CnqPOP=v!+U}+IPv+vvU,C5OCIIPO!IO-V=d!UU5+vU+i-vvPn5,C!,BCn==n=>-{iU+==!qU++C,IiCCI,nn-,U--C5MOnC>i-IU=FvP55UPiPC>=PU5Pv-InIR>i!-Ui?)!n5!+vPi5-CI5vnOI,-nO>*<O=>UI5U+3eU-v!vOUOvOC>5>v+,CnU=>-OOnD,-v>P!-5!n,=!5On>I,5OCCIiO>=!-UOa{=i>5PCnIinnIPI>>,==-n>i!U>=v-i-5>v=,,n+,+I-Ov=n-,>!m_>n+VUiiC!v,>+=CO5-O5,>-C=II+UC/U!>>5vOUC5nPIIUvv,-ICC=(CO!+,-iiPYvP=+U!C,iv5Cv5OO-,#UP=U!!O=EOPni>v7i=55C3,+nU=I-,=+!POO+!- i-!P,I+5C>iCnn,i5=O5In-V=>!C>!vIUi5nPnI,n!=Un-O>IPnO>v-Ii,{U!=>+v-iP5>,IICn+^!n0>UIvO->C-=iI!v,i5-CO5nnU,,--Cn&>n=>5-5i>+nPC+vv+i>nnPvIUC,cUnn>P-POv++!C5,!nP>n,v=,nni=Un=O5,OU>>vPP>5!IUC+nC!,nn)=vIv=I*>U-C5-5iIv!!v5UCP,I5iP=IIn+I!-COOV+iP>+PPiiv,iI+Cv7I5nICCI+O>-IU;C-,i5O!>,!5,C5iC+i=!InO_Rv-vKI!Cnv+=UPni=O-v>i!,Ii>,C=O,O=3i>I+n-+iP=iP-nlviIi>IC=5PO+lIU>+5!5>=+U-5>vDN,OnP=n-UOUIv-UCUI+nC+5!k5iCI,Iv>CiiIv5P+-UOC!-iP+PUniO%PUn>OCI,+O,=%Ib=iW+-5>O!eUO+bP-5!PO,4n>,,I,C-,Cnn=I-IU++IP>55v5i=5=!5IvC,)!n-=I=+OI==-OUU!-,C+,=P5PvC,nnPCv,U-&>5!=i+++i,i^c+iI+I=I5i>O3n5I>iZUi,>OIi>FvvPn5>=!,>O!=U-P=,IvU,C=!n>i&+i55Cv_I-v>=,I>>P6=-=*U!>Ui+nP=inv=,I5q,nI=Ci=Cnn>>-PUv+>U+5PPPPn5DCC,gCn=P-OOU=_-C>vUUi,!+,P5ICnUvn-P!IvCnZ!5=>/!Pii!UP-ivCII!n!,5IiCP=>n5>+,vUvW5->>OvnU0nPPUIPv+=v5tvI=O5i>5,>n>=OP=+5!!,>n,=O-iOiICU,O=!!ii+!Pii+v55!n5P=5nO!=n-l>v^v>I+!-Cinl,,-+5A+,P>=C)i,v,,PUis!!n>,v-Uv>=!=i,v5P^-PCO,5n+Cv--OCyPUUO=v{i5+nPviUnOWPICO5hUU-(U!v>OsvU->=Piidn5PU-iOn CnP>+!Ii>v5P5+=v!U55+CnI!O+,!->>,!Oii+iUCU6#iU=vOCnI!O5A---=+!!-v>=P-U=v-PO5UP=,Ovv,iI=Oi#CU>>>UPi5=>P-+U!4Unn,C>-PO===OU>,,=Un*OUIOvviP=nU=,I,CO=!5,CO,>U->vPI5!v!i55!!!5!n5,iI+Cn,=n==I-!>,?vUO>>COUt++Pt5>CIz-5O>IR+i,+7!:+ivv-35+P>,8vv,P5CCCIPn=GPI>i-+v,In!C!55n-P!n!C+IPnn=+PPU= CP,>e!IUpvIP=i5O-&,n+>O,vn5=6UPiPvn,i5n,PIIOP,>I+v+,i-UL!!!i5vUP5v!vOIP5CC5IUO-IC->=OI=n=+9PIi5C!5!ni!O5!vP=I5UCC,CU=+!PUOOv>P=5,C+I,O!,O-5v>,Un+=nIKO5!!P!55CU,5C!CO-PnC=5-U>--UUi=i-P>O!+,=5v=P5nnC,vnOC+InO,V--{i-+viP5iPU5,vU,P5+CCInnP+,!-iO=CPP5OvUPc5CCvnUO>,CUv>!POnUC5I+5,CPii5+PnI>vC,-n!COIIOn},-aU=!,i!5P!?5Pv,2P5UCII5O,=CIii5+OP=>,vv,-n+=nInC^kn5n>,I!U>+,PO5iviiCn,v=I!Oi=!-iO+g5>!+v-=iI!IP+5IC>I5n5,=ICvOZiU-=C!,Uv+,iPi=!=P=nU=IIUC==+55>C-,Ov=UP,5!!OP=>>!UiCv?Cw-i>-KiO<>nP!Uv+iP-5IPU5!+5=nI>Oh,IU->i!+nE+O-I5U!nPCOi!!-O+O=>Uv>+-PUOHIPU>C4kUd+IPn5!O,,>5nCv,COU==!i>,lOUv>UCii=nI,II+OI&>U5>5-=i-=5UC+nve,P5O,iII+%,_nP=,=hO>Z,Uiin!8,i5i,!Ii+O=v5=COYvn#>-PCn5C!-!5P=II,vnCT5>OII!-O=O!nU>+%U=i5v8P+5UCII,C+oP5i>5eOU==,!vi-v+,n5nPQ,P+n,C-UCn,!->>5_+ii+=Iin&PPPcv5,I-Ovv,,nn+UInndM+U->/+UUP+n,-IIn+D,-sOf-iUvCm!OO5vPPOn!CC,CC-=OIUO5bC-5>C!,U=!5Pi5nP!iUv!P+5+nCuU-PO+.OUn>i-=>O!-P!+vCC5!>O=OiOO!Pn5nvO,>>O!Ui-++PPi>vU,!n!OgEPUiEU!-UvvI,!5!P5,v5nC>-!n>)!-U>P->UP++UI>CvUPCn-=PIPCn=n5Pn==5n->=!iiCv>P>vPC5U>55C!iCO5=uUi+I!I>>+C-Ii!+OUin!CnIcOv=vOIOO,v-O+--=iC!-PC5CCU5!O,CU5,Ov7-U++n!n>lvU-n5,PIiCnUCC-->P7POn+,IP-=>5POi+vI5!5O!O5!C!T,O,>OQO>I+,!>5Pv=P=vUC-U=OI,OnIvvdi-=+UP,i,!OPA>-CP,MvnCxiOv-,+n=O=!UiI+UU=5-e5i5n5PC555CPOn=OP=>OO=,PIi5++,-5+CO5,nOPIi=C-In-n>:!CU3!nP5ifCiIInI,>I5vI=+n==nIPU=+vUCU^PI,C5+=!5(O-,5->vnP5UC>F!>i!=Ii!5CP!Unn,C^IIO=Dii=v5-OU=#=Ui+i!>i+O,PUivC-IPOP=O-,U,+OP5iOP,PF>+CC5!n==,i++-m+->C,I!>O+=U=>5Cn,>ne,=--v5%P5Q+!!-UnvPPn5i!x,i+!!+5!CU=U-v>>*v>U++I=55!OiI>vCC,QO-Pn-C>=w,i,CCP!OUviPn5C!PIPnIC+nnO+,Pnn=n!P>,uvPvivPi,U5C=--POPIn-=CP!iOvbiI2ivv>iI5>POI5v+,PnPC5I>nPCv-->i+iPC5+vC5inUCC-->P.POn+!IPUC}vP>>=!Ii-+nPCiiO!=n-&>v4v>I+nIvin!5,U+O!viyv!,n5PO+^IU>+5!5>=v--5i-CiiCv5Pj-,COIIn+>-!-Uv:5P!5!vnP,> CvI>CI4I5-v6I5OO>OP!i=v!iO5o=-,iOnPI-5C=InnP+,P5U=v+-i5>PPi++-=UIOnCyi-C>+--i-=U-!+U+nUCivP+,+O,s!-,=+!Pii>nP>OU+xi!+>!II-nnCv-UOv7>OI+II-nb+-,5>+C+i>5>=PI8OPI>U!+UB5iO=-!=>B!Oi!v!PP-!vnP=ni>U)O-C+i!Ci+!-,->U!!,UOnP+-vC+=+U,+!!,>+vP,iinC>UU5#,!5>CPIPn,+PIOnH;5Pi5+v,,#>>C=5UvQPn-OO+!!n-+P!D>n+8U=5+P!iiviPv5Jvi,Pnn=+}+i,v!P,++CPIi5>=>iUOvRC-=+IT5i5v,PUi5vvi+5CC+5-n+,>-5CvIPO,=5-+O,=CU-+5v5,=nvC=n5OC!,--+iIPinvOP>5:vPIIn==PIIOnI5-O>5-!U5qiPI>n1vU=+,P5U=+O,!nIOIe>Un>>UIiOv=PUn,YCI-OU=i-+O,NDU>>C)Wi-!IPI5=vvI,>C=!iUOi=n-C=v!,U-+OICU=CIUin5!,I5vC,55!>!Z,->#5!CO!K5U55!PPi+n+C+nUOn!!UP=--->->5-vU+!>IP>vC!I>C,,v5U>isnUC=P!UiC+OPI5PC!5O5<P--UO5Jvn!>>!,iOviPi+CC,P=n!=iI!Oi=+-57!0OiP^OP!>+vCiP5>P>I,n>1P-=O=-U-=C=PPOiv!Pn53Cv,vCICvivOi,--n+!RvUi+-PI+vv,i>n5P>IC++!>U!>,NCUICiUI5!!I,i>-rIIOn+=iIv+2IvO5Oj!/>,CU,-OqC,,Pn=CUi,C+M,O!O5-iiqtI,I>aCPinv!CO5OC-8I-U>>-OUn+!P55-v-i+n!vv,=O-C=--OOLUO=>i-vinL=!v+-C!iCnUCC-->PkPOn>C6OU+vP!+5Pvi,,v+C-IvC-,On-CdIoU5>KPi5IvIi>n,P-,iv>=Cn!OONCOP+I!=>,+CUnn!!=P+v-C+5>O-I!UU=iW_nC+5!o5iCI,Iv>=!iIv+P+I>>-r=Ui+n-+>, ,,-v-C+,+CU=5-U=i,=5=+,U,UOv!P=5!PO,+nO,>5Uv,P=UP2PGnU +C!*+nC!UPnPPviOv=PL5>C7IUOU=U-I>iviU&>nvOUvnv,55-CO,>n,=>I_>!Q-U!+PCPi>+I=-Innv0UOU>5!U>ioP-!5,P,POn!C=I!CO=,->O>!!U.>5Uiinvii55C+;U+v,,=IUOv_>-vBU!-n=HO-C>+}vII+!P+5,vIB5-,>>!OU>=UPiinvCUPnI!UIUC!PCn,C,I,OI=5-OOCj+U>5>PIUvnC=ni+O5jOOO>i!>nC/U-UiivnPCnIC+n,nO=iI=>O,=U0+I!55!P!,5>OCO5-vP,i5iCiI5n+=C-PO&A=P=+5!!IPnv!RI+OCIC->>=IPOO=OP>ivCP5!niCnICCvM,-iO>!I>I+UPI+-v>-=+!,!,5n==vI==5?OU5yn!=O!=vU5>=P>,Pnn=iIn=Phi-n>!-CU>=i-v>5!nU5+I,UIvnvIi-n>i-5nKOSI+i=!=PU5vC>,vCU=Ii=>!IIiv>v,vUIC+I+nv=Civvn,55=C-,Cnn=I-IO>EP-,5i0iPIiv!OICnn2!-p>!,+Uv>=PIO5+I-+invPU=n-P,,uC5I,nCOOIii>+UPC5vvCUnnOC+-!v-qU-OOC!i>i>5I_Uv+UU,5n!i,Iv+,inPnC,>U+>CP,+Pv5P+iHCn5nn>=nnOC,,IUU8UJviIvPPI+vvU,C5C=I-,n>IO-+>O->i!>,I=>!PPPn5oCC,6Cn=U5P>PIvnnLPI>i=+O,Pn!CPUvnCC2--vn9O5v=5ICna+!P,i5vC,UvvCII!nnaI5n>O_CUP+>U>i5qI,I+=!>5I+v=CIH>--IUO>+P!>wvUP55vP+IP+-=55vCP,,UI>U!>n=+=P!inPU,+>=Pi5Un<I5-=O=-nU>+nUPi*rP-CnPP5,5n==vI==5=ni,+I-ii>vv-5i1C%I!vUP>-Ovif5-+Oa!n>n+U,I5-C5U,5>=>I+C!,UU-CP!,U-+OICi=v!,U>OCI,+O,=&Ig=iHv5L>>-UiO{CU,+=vC,Unv=OIO=!di5+>UMCi-vPPP+nv.UI5CCUIvOO=OO!>U.PUI+O!IiO+=P>vICi5Pvi,P5vCv=OUI>C!Ui5!5!=i+vI,=v==UIUCGEU-5>v-+iP+IPn+5++,U5mC5IOCO=!ICOU!!>!+5!5+PvI,Pv,CPUvviP>5O=-=+U,+!!,>++>--5!!vU=++,nI=O!1UO->>,Ci,vU!85Oti,n>>CC-,n=Ni5I>!J=O<=nICO,==,,vPvn,lnCCFnnO>anOO=II+nvvPi!i5v=,v5=,5IOO5In5CC!,vi!sz!iiCv+PCviCnIiC5==5Pv+IPO=>U!vi>+viU55CU5ivPP!-I=,=OU!>=!!>O++PO+>CPP,>=PUnPnn=f-COr-nUUvIP-5O+C,Cnn!I,,O+=vUPOI!Ui!CC!=i,+iP:+Rv=,inC=>I>=PK!5>>CP,UiviP,i>CP,=5=,U,v+=C+-I+P9+iU_U!>5nv,U=+I=,5P5nPUIoO5c=U+>+U,U+=+UI>ICI,U5C,>-!vI=%n>>vPPUnvUPOiOPIPv>vC,5-vOP>-,+!!Pi-CE!I55++,+>nCO,+O!,%-UO5^vO++P!UUOv,i,5PvOI!nCCCn-O+P;-5+U!ii+>VPe5>tUPjnCC=-InUr5U,>b!!iU>vPO5-?PP+>UCPiCv,PI--v5avUZ>OPIO!+n,i55Cv,!O!=+iiO!K=-2+-_iinvI-!i>e-,!+vPPi,OIPi-+>=Gni,C6PPOIvi,+iTCNUvn=C=nnOP O-UOb#CUv!UP,n!C,I-v-=i5>np,v-a=IInOn+>P.iPCIi,+CCv,=OIP5-Pv+)5OI=nIOi>=C,,nUvrIO+i=ni>OC!,-i+iIIiU+UU(inC!Pv5iC-IICv6in>OaIvU4S,P>>+v>PvnP!U,Y+O=UnPCi,5UnC+P!5I+C,5>-v=Unn+g!I,>-,PU,>-!O>n+=P,iivxiMnPC}n!viP>5->n-5-=+UPIiU!=,!5=P^ii+OPI-5=i=CU-+,!->C+iP=i=C-II5+,>IvO>I+nPOI,6iI!,!O5!v=,!vOC+IOC>=5-PO5-iUIvPP,55++,+niv>IPng=Pn>OO,I-y=+-!OPv,Pii>CI5In,!vI<>-=iUnCU!iUn+C-P5Uv!,nn5CniIO-=i-+vD:5iU+iP+UEvz,>>UvQICn=&IIU>5!,U.+!PUUvvO,->Pv,,-5OCpI5C==U-,O>!Un>++!wiIvviv5=Cv5Cn>=-I>=O}ii-+UP>U=v=,OiCC-I,n-,CIvvi6,n==II-iU+O!C5iPi,U>R=,-5nOd+5n>O_+i!=-PUi5vvi+nPCU,OO,I,-->,-IU>C+-P>N+iPC5+vC5inn=in5O-==--hI!!i=+u,-inCnII55C=Ivn=I5-iC!fvOn=CI=i8vIP5n!,!,d+O=vUPOI!Un,+I!Ui>==Pd5Iv5I!C!CC-+Ov!P-U+UP!n+>n!PO=va,Pni,UI+nCh,OP>52OU=2CPIi5++,-v-CiI-CU=viCvC-P-n>N!CUd!nP>5nPOI!+P!C5-v=I+-,>O!5UO!,P-5,PI,>>v!5i=v5IUIv>I!PUIcv!+iIv>,555P=IP+5C>-N>!!U-5+OP-OP++PI5>C5,5v==niOOI=+U,>hmz>i+CI^Uv+UU,n5!iInv+=5i!>ORPUP>-kj5n!-!viUPPPCO-=*-!OiL+UUOCIIiU+PPI5OvI,O5=C>5+O!,C-z=ia=Ui+CP>i>PP,5>>C=-,OI*i5IO5(o-+>U!Ii,!+,Pni!+I>O5=n-CCP!PUvCO!>UvvPU-5vC5,In5C!IOnrK--=Oi,vU=>>!v5,+v,,55CI5-n+,i-nCv2UOU>C!UivvOPOv!CiUOnC;P-,>ULnUC=,PPi=+1,->nCnII>=CniiO5=O-=C,=+O>=IPIiU+Ci>5I!Ii>v>=vUPOn!UUO>OUIi,=vPn+-!OU>n-Cv-I>!J!O5>CI!iUvnP,5vLO,in-,UI+>!=vU-vM,>O!=iUnUyvi,-5iP_IiO>C=5=CU!-5HOv!->PvI,ni>CvP5+-=UIOnCliOi>CPPUUSU-v5+QiPI5!PnI>Os=PUIn=,+Uv>=PIO5v+,Cv!Cvi=vIM,iC>P,UUi>n!COP+O,55nCC,POP=vi5nvI,-C=-!>O>+iUIOvvC,P5UC=5=n=mO->>TYPiI+=IOiICP,,n5v+I+OiPI--Oi(+O>+!!-UnvPiP5ICP5,nO!>iiviIO-!>5!UU5!!!O5PvP,5niv=5Cnb=Cn=>U,nn==OUUUvvI,P5IPv,OO5=n-COP!PUvC5V--C=>P=iOCPI!nP!vICnN(-5n>!!=UJv-!n5nCIiI+C!i,5nOC=--Ov-I->>5bbi>=A,!5-vnIPCPC=-vOC!,-i+iPP>P.+--5Uv5,vv+=PIIOnI5-C>PMUU=Q=P!i=!RU,>O!II5CiCC-->,w-OC>q!C>=vU-i>,!i5U5v=I-POIIvUP>IQ+>n+5-P>i CiI+I!=n!O5=5OP>I!P>,=-_>Oiv>iO5!C5IUn5I!-5O5-P->+,&=Un+iPU+=C-,in+!x-In=wiUU>iIPi,+-POOCvUIIn-=O,COC?n5IOnI=-vvPP,55+O,+niP,,+n+=I5=O>I5n!+-Bri5viP5>,CI,Un>!=I#OI=5U!#!1Ci++v,PiUCUI!>+C!5in-DO->>xtPiI+=->iIvIP++iC,iS+n=OI+>!I#UU>OdCii!iPInPC,I55+=+-ivPIP-h+-ziin>,I=>i!nUPn,C-IOCn==-!>U--U>>vPP+!viP>i=C555nO=5nnOM,,5v=,I*>>+PPn5ivn5PnI=Pn,vv,P55CP!U>->+P,5!v,i+5CC+5vviPOih>--I->+P!qiP!>Pv5>P+iU++v=i>O=IC-->+!OU+!-POi+vI5!nn!O5!vi,+5CC--nU:>HUOi+vOi>nPv,U=vUPOnvOIq>Un>>UIi!v=P*n-vnInOI!=In>i%5UvO=P!i+=nPvi5vW,=5x!>I+nC6,5i>I!PO>>8PCi=CIP5n5=,Pv+n=+U!O5!-OI>I!>in+>iI5,Tv,nv-POi>O+=0-I>v-vU+=UPn5C+sIP>=C IPOiP>-=OO!Pi!+PIviC+S,->nCO,CnP=>n>Ov9>OI>nPii5vvP!n!C+PU+P=--OO!zCnvOv!IiP+IUvi+6U,P+CP,iIO-=nIv>U-UU-C=PP5i+n,>>5Cn,>nMPI--OiT+O>+!!-UnvPiP5ICP5,+vPIi5O>IO-!>5!UU5!!PUi5+&iv5+!Ui+vU,555Cn--U+>+UUi5vUii+!!IU+n=,CI-O+#O-+L-!Pi%v!,UiOCOI-OP=,-5nO1+UiCP!-iO+!PCO>v=POnP=!IP+v=CIZ>-,nU!+=!G5-+n,nnI(=PCOi=5-vn=!!U+Cn!vU5+/P=i0*>,+5C=,iiO5=+I#>n-nUUvIP-5O+C,Cnn=UIiO+CC-V>>,UUn+C!-5P==PR5PCi5Un+CC-,=P(5-+O{!n>n+>Pn+O!IU>>=!nIvC+=,-O>59O>,+-P,+Iv>-++I!+5>nP=n-iOn-P->+,!,inv5!x+=C!,=v/PO5!v-S5OiOC!-i,+-UC5,v-PvvO=!i,v>,O5C=+!,U,kv!=iv!CUi>UC>5On!=5-UO5-!U,+!UPin=OU,+Cv-,+nOC+n-O-o#U!+Ug5iOv-I0iOC5,nnCCP-POvPO-IvCdOOU=>I+iv=R,-n5CPI++n=OICOPu>O>>5PUiiv+!c5%C>UUvU=I-nn>9vIiC,I>Ov=U!zOOvUiP+i!5In++E!UIOC!5n-+U!5iv!+,P5UvOI,C,=--,=IA>5+=5I=>>+PPn5ivn5P5>=,I,Onj5I_==!!U=d9PiOOxCi+5,COI5nOI,IZ>Cz=iI>5P55,=CP>nUCiI+5C=/->v5jP5+>5-IOn=OP>OCC,IU5a=OiiO>=U-C>vtCnn+O!+5!&-,P5/Pn,CO+=vUPOU!Ui!O>IiiOv=PUn,PP,Pnn=iIn=Pj!5>>i-,O5=nPOiCvP,>v>COiIOi6+IC>D,vUC>ZP-On+OP+5!Ci,=CUCvIOOPFvnP+,!iU>vIiI5UCIivnP=S-!>U=OUO+-==n>+C,,iOCiiU5UCvI>nvIU--v=D>Oi=+IviCvPPU5=P=,C+5=>-YOP!In!+P!Iin=vPCi*C-5InOC+-!CjbU-5>v-+i->zP55iv5U,nICUI>+==iU->U!>-=+=PO5iv5,vi==!I++i=O-=OU!,5o>!!,U5+CPU+vvI,!5n=IinOO=C-P>>->U5vUPi5++r,zn>=5InOCCcUP>v,5U>+2!i5I}!,P5ICn55nC=PIUO=I=-+=v!>iW+I,I>cCPIi5n=>5Ov,1I-U>>-OUu+PPi+Uv+P:5ICv5vn==vnC>-,Unv=UU-U+v,,!5,P+,Cn+,v-Ivi,+nPKI >iP+ PP+>vv,>v+=,,I+w,Un!=Cl-U++O!++-+C,U5UC+I>n,IP-I>P-,UOO>IiOvQIi-i+C,I!n,,+I>O,.OUi>i-CUv=iPO5=v,I,5v=>I-O=,=U!=-YOOi+P-C>,!,,-nOv+ICvv,5-OO-9vU+>vI5in+>Pt>IC!,=vYCv->O+!!--+-!yUI+>Pni>PI,,>vCn5-vOP>-+Oq^IUv9v!+OUvn,CiE=PU=n_=P-iv>=+-C>,!ni!!5!=i+vI,=+I=-Innv_UOU>-qviIv!P!+5viU!nI=n,vOv=iU-O=!nOn+>-=iI!!P++5!vivn=cIIU>5IiOP+!!,i5!iPv5!v-,CvCCJICC=#U5iCCI5>U>vPI5PvIivnPCI,+Cn=55PC5I!OI=--P+!v5P5vPCIIPC,=O,>vi,vn,M-r+i,v!P,++v>,,nO=iIiCC=v5i>O!=U,v,IR5PvPi+5-Cv,nn,=!I =nzUiI>CP5>5vOUC5,!l,>viP+5+OC!,--+i-U>!+EPP5iB>,=5O=P-!OPPv-COz!-nn+!P=i%C-Pnnn=IU=nn4i-5>v==i!++Iniv+5P85=v6U>n+CC-,viV5-+OF!n>n+5!R5iCI,Iv>COiIOis+I4><,vU=>=UniPvOPUi?vC,vCU=,U!O>!-O-+i->UhtvP5+I!ninn>=BIP>II,nC>v:=iI=5Pni>vli=n-CiI+C>D!--On!P>P+IPP+,vO->>i!viiC-C+-,>!#,O+>C!+>vvI-->!!iiIv=CUIvO>=vOU>>zvU-!PP!O>MKUi+v!>ivCO}!-!=>&vU>z+P,OID>U-v,vOI!n==!nOOi!-UU+>p=i=vO--5PCv,CO,C--i>PP=U,OC!Ui-+U-!5PvI,n>vCi,=OU/,-,=OSnn,+UP>U=v=-+5CvC55n!=nI-n==v-+k-!Pi4+O,I+ICUiO5=P+IiC,,5n5>O!=U!v,UP>vv+PCn,!iI>nU=C-vOC,nUO>+P!O-vUPOiCCi5inUCC-->PZPOn>5IPi-vO!C5C2>,v5v,i,cO5=IICO+%>>I+!P=inC,i,n-Pn,Cv>=UnPCiIiUn+C_q5P!!U+5>vvIP+U=iInOCIvU,>i1>iI!IPU5IP-UC+n!5IvC+=,-O>5BO>,+O!O+I+v,-5!v>,nn5I!-iOncCnP+,!iU>vIiI5>v>5-5C=UIPn+=O-n_Po=iv+C,,iiCiIPn-CiI+C>E!-,>5-iUv>=PI+,vnP>56P=I-ni=+n>>!E,U5&i!vi!+-PC+CvG,Cv=PU5PvPPCUPY!Q5U=+v!=+5vO,5vnCYi,+v,P5>=>MPUn+i!n+PvI,Pv,COUv+iP+5n=-=+U,+!!,>++CP++v!UUO>?!+IdC==U-v>>8v>U+,,!5PCiP>n>=Ui!5CCi5IOC,5UI=;--OUvi-O5==P,+OUP,I>>n(OU=>,P,iC=n!5n!CPIi5n=>-Uv,=v5i>,I=OI=-PUOnvCI!5>=-iPnO{5-n>ClPiP+vI5iPvB,!nUv5IOO-bP-,>5==U++iIIUC=5PI>b!-UUni!OI=>P=+UUC,&>in+OP=i,C,,C>nvUP=++=Ui_O+I5nvCC!=OPvi,O5ICCU>n,7!-P>i=>U>+UP,iIvn!>5vC5U,5v=>I+>!=5U->0,vUi=!!v>npC-=5N!,I5O>=--=v+}IiP+,P5U+v+,i>Pv!IOn>=xIP>I:=5+>U,/U+o5-vOCv=UPni=OIIOCP>-,+!!Pii>>P>5U<!P>nnCOI=n!E,-C>>T+i!>5P-iE=vPC5PvU,=v==!I=C4&iI5CIIOng!P!niHvCP1vnCPIOnO=N-=Oi-UU5+UUiO=dv!O>UCO5n58=i--OiIY-=>i!Ci>+>iP55j>,CO,Ci-ivIfU-U=<bni!>v!ii-vIivniP>,:vv==n,>>I+U>>vPPOUvOP-5vC+,v+5=nI>O(,I-+=v!>i*+I,I>hCPIi5n=>5Ov,=I-UO>!!Unl !iiI++,i>+CvI!n-=CnCOv4-U++n!n>avU-n5+=!,-O-PP-IOIIC-i>=z>U-+,PP+>C-in5CP>IvC!6nnO>no>U6=IP-iiv+i>n!C,I5Ci=v-!O-NCOC>q!C>=RiU!>PCi5U5v=I-POIIv-=>v-COi=>-,5UP-P+n,=!I,C+N!-,O>-5Ui=!-5OC!,UP>vPEIiniI!-,>!-PnO=-(-O!v-iIi>CP,_nP,>I5>UJiU+OW!?i>7>U-OCCPPvn-=II-+.9!-,>5,+UU>CP-5PvPin5P!PI-OOCC-COOcPUn+U!U>v+I-Ui,v>In5v=v5vOPBm-I=UI>iO_nKBOvviP=nU=,I,CO=n5,C>,>U>>v!-+Pv5->5!PPIIOnC#-v>!%!O>>OIIUmb+U!>PvOI5nn=+iin>9=-n+,,.i!+,P5+iv<P>n,=PI,+C==-!>U,OU>>=!,i+!+P>5,COIini,CIiviAOU=>,P,U=+iPC5>v>5P5n!>,5nEmC-P+P-PUiv-!n>>9Z,=+Cv-iPn+=I->>5M5O=>CI5O6=RPj5PvO5inv!JIUCi)nUC>-PPiU+UUzi=3n,-v!PUiin=Qv-C+!,+-QvI!C55s-,U55Cv5+OP=I-n=5:CUP>U!=>=v!P=+tCiU5+CPnninC_-U,>--Ci,+-!v+OvnU,+5PP5-viP#OP>nrn>,+-P,+I)+UUii!,IiCUCv-I>PkIOv+P!IU+!nP=>PC-IO5C=C5OO+!!-,+-!nUn!,!F5Cv=IP>vCiIin=P>-+OC!,ni+>!UiCvvPC>nCO,+O!P-IF>>(+i!>,P-is=>P-n,CIIn5v=v-5v-gn-I>+!>U+=iP5iOv=U,nIC5,+O-I--OO+jI>!+iIOivCPPUnUPPIIOnC>-v>!k!O>>5PUiivO--i=v=,i+,=IIUO>P=-V>P!i>U++!C5,PP,55OC=5COI=5I+>---Ui+-UUOCz,-n>-Cn555==U-IOUI=U!>=-?ii=O-I>nKe5P5nCsICn9In->>n-Oi!>PIC>,dvi+5,COI5nOI,I+>I7IUO+n!!>dvPP*v!!ni!5I!F-I=,=OU!>=!!>O+nP!55C-,-v+=Pi-O59v-!+!q>in+OPvO5vP,Gn!=I5Innsi-5CvIPi!=RPPUvv-,I5-omI!n,=5i+OC=CO5>!!nU->=!vi+P-,Pnb=!-UnOWOU->5pOU=sC!IiU+>,!5nP1,inIC+-iv+_vU!>-!C>C+vP-5+Cn,nv/=UinO+!!--+-y=iv+C,!O+vUIIn-=555nv(>-+=!IUi-EIP,i-vOin5=C,,in),s-PO_-!U5CnI=Oi!5!=5UCI,Uv==IIUnCI>-iCIIOOUyU-i>-P,,O5O,IIUOII-5vCP,5Uv1+!,iOv5POv,v}ICn=QII5>5!,5C=C!+5!+,,-U=F>i,v-!C-PnvE-UI>-,hi!+,P5O+vUPCn-=PIPCn=85I>IoU-CF>!iOI7+U>>CPvIInI,CIzOCI=n5Ci!!>>+PPn5ivn5PnI=Pn,vvP+5iO3-OU!+5PUi5P!,5>OCv-PnUKUIM>Cw1iPCv!i5-vi,n+nvCI+nC,P5i>UI--vCi!,O=lI--5Ugn,CO!C>--vPq,-iO>!I>I+!UPi*C-PnnnP-IiO+CC-_C=,5UPC+!5>I%n-O5>mCI,OUC1-Ovi_5-O>=-CiI+5!+5-P-,in-,UiC+C=5OPOn&fUC>eUniUCI,-nOvCICOnPI-!>+jviP>IPU5!=CP5>PvCiO+=!)-!vIpnU+>U!;nv+CPPiUv=i=n!C=5,vOC55IOv-i-C+-P,i-!CPi5=v=I-OIC+n>Ov9>O+>5xIn7oIi,iOC!,=n!,OIiC5qUU>O=!=O>+v,PiICUi->CC5iPnC,O5=vM!!nI+nP+iUv1-vn!v+IIO,=Ii=O<dPUiC>!vn=vP,ii>C>,-O,=--ivP=+UO>+!COC+,,!5,!iU+n>PO,nn>Ct-UOC---+>n!!i+_!,P5UvOI,C,Cwn!O=!I-5+5-IiUv>!v5=!CUin!!>IiC,,55n>O,viPv-!=5n;U,i5nCC5vO,=--O=n:=U!+UU-i>+v,Pv!CO,-nv=+Ivv5tn->>7IIi-+n!v5UPU,,O!=P-in>4>UUC!-!U=vI!U55+P-C+UP5i!OP=I-n=5/C-o+-UIiO+CPP5>P>,vn>,+5,vT,!5>>=-CU-++POi+P-,in-,UIv5CPO5CC+-5-=+UPIiU!=,I5UvC5>O,PI55voIUn5=PU,iO+OiI5UCI5-++P5,5vI*5OiOC!-i,+-UCivdi,On=C!-,+umII=>i!UUi=PP,i-vO-C5>Cn5Onib--U>>==U=+O*Ci-v,P-+CvvUin,P=5Iv-ZU-OOC!i>i+UI:5,C5POn+!nIOn+.!5->n.IU++>!+Oiv5PO5=!,IIn5C+--=-?iU-)U!,5!vP,ii>C>IU5O=!I=O!IO-nC,c=O>=<-!5PvUPOn,,,IP++==UIOU!5n-+U!5ivw!,P5ICn55nC=PIUO=I=U!>=-Dii>5-I>n{*5P5nCYICnjIn-P>OtOUY+=!i+Uv5,UviCCU=vI,,nCO-A+UO>+U-iI++,,5<vj5inv!3-,>5=+U+>UPIi-v5-,ivC>,+n=P=II>P^,n5Cv!+O>+v!5iFv=Po>>C+,CO,Pi-I>PI>-7+C!=5I+5,5n,+vUnn+Q!I5>-II-I>>!nU>!IP,Ovvni-+O!>I+nF=I-v=vN+nU+nPCUMCP-=5oCPIi+>C+ICO,3nU!*5%=U++IP=>IC-,n5v=UnUO-=vUI+!!!>5+C-!5ICnPvnvCi--OUVn5IOC!+Uv+_-di-C,,I+n!CIvv+=>Iv>P-!Ui>n!C>vv,P-5OPn,=n!=Un-O>==-,>+-+UC++Uv>,!P-;n-,I,>OP=x-P=>pvU>7+-,O>==,Iv,vOI!n==!nOO+hOO>=PIvnC=OPC+vvI,>nnC>nIO,=>UP>=L=>U+-I=5PCiP>n>C--,nv1ini>nIvUP==!O>U2>U>5v=P,IOU,-5QO=w,-i><-BiP+ui!>5!UUU>3=In,nOd!-=>!-OU,+>!>5!vVP5viCnIiC5P=5nvvkwO=>U!vi>+viU5>vv,-CP=5i>OC!,-i+i-,i-vO!+5CCP,Pv+Cn-iO5z>5UO74NU5=IP-iiv+-)nIv=IiOU=i5P>,W-UOCC!U5Iv-,OiCCCInOU=i-+nCw9U>CUZBiC+=,Ii>C5I,>8C!I,n5=C-U=v?IU!>nPIOnvOPC5PC>5>n58U-i>+=rUA+>P5invC!.nPCvU5nP=u-!>U=vUO+-IPi,+-PO+nv=,!nU,-I>n==,-+=+qCU+xv-IO>)P,-vIv>IPn;=Pn>OvE>O++,IIOn==i,iOC!,=n!,OInO!_5U->--+iP=-P55vv!I!5>=nIOOv,v-P>J!!OU=>PO>nv5P+i^Cn5nnU.I-->O=CUC+nPUiiv+!C5TC>UU5{=CI=>I=>U5+,,Ei!+,P5+ivv,!5-CC5CnV=Cn=>U=in,=iI=+!+5P=5vv=55niC=-U>,},OO>vI,UUv>P+n!v,I-ns!vIin=HUU,>,-OiI=-!=iivC,>5>,PIn+>C5I!vC!,nP+I-ii,C+,-ivvvPD5>hI5^n5=!nvOn!ki>++PP5iC!Pn>==!,vn=p-I=>-LOUU^i!+>nv>iP5OCPInOU=Unv>,,UUO+C!=5P==P,5>vi,!5=CCniOv!Pni+UP,iIvn-v5vC5U-nUC5Ivv.t5U,O=!,-++-!>i*vOPP>5CO,Un5=CI5OCg,-==<!i>PvIU55!P!,nn!=5--O-I+UPC-!niv+C,!5ICnUCnv=OI>OB,IUI>=,OUI=PP,i-vO-CiiPUU=n==!In=U==5==U-Ui5vvPIn!C-,-v=CCi5OI,2n-CU?2U5+=P+i+P,,n>+=!-InCk55->P(u>!+iP+i5vk->>U!+iPCIC>-PObrPO>+PPUUO{OU!5fW>P55.PvI=>I=UU5O,,}i!+-!n5PPP,nnvC!5!v5.i5Pn==+OI+UP>Uvv=!O>iC5,On=P,-i>nI+U5=OI=iC=nPv>!CP,Inn!vI->,oIUnOv!vi5=,!5>CvnUFnUPUIPv=P5-nOv;!UO&O!ni!v5,-5-P+I,+iC=5MOC!,-i+i-,i-vO!+5C!vUU5r!OIUCP,i55>n,+i!vI!C55Q-Pvn>C+-!n-*--}v>A-i,+IPnU>vv,5>-v=Unn-,!5Uvi;55>>JP,Uvvi-I5UD5,>n{CI-InC%+-C>GIciPDU!>>5v>U=+IPIIUO>Cv-=CCInU!C>!i>,65-n5O{vIPO-C=-nvU?U-5O*-vi,=UPn5CvPIP+CCu--ni_n-vOv-iUIvPP,5UA!P>5>C,U=n6=P-iv>w-5=>>-iO+=vPC>!CUInn,=viOO+=+OUO=!iU,>v!>iOP,PQnCC=-In5z5U,>Ub5Uv&+PPiIvni55CvjI-CI=OI+>!IAUU>O/Cii!iPn5iP5,=+P!+i=v>IO-!>5!UU5!!P,5!PPU5+-!ii!O-III>>PBKUP}>!vi>!+UP>CTtU>n=,CI-O+TO-+K-!InCv!,Ui5COUin5C+,}OnIn-5Oq!iiI+IU>5PFU,!n+Cv-PnI_UU!v+SUiI+-POUCvC,n>UvDUOnU,P5iv5Yn5++!PIUCv5--iv!C,+O!C---C!a,U5OO!+O>=I!CO5vIUj+-!UIi+O==UPO+!Un,>>PniOv=P,n,CCUnnU{!-P>i=nU>+UI,Uv=iP,>=!IU-nU!nIC>!=>U-CP)Oi5+nPCiPCP,v>5CPI*O! UI5>O!-nP>+IUiPrCU,>IC-U5nv=sIO>I,!U,C-!5iv+!,!i>Cn,>nvPvIPOx9PnUC>!OOn+!I>ii!,U5>nCOUvOPw-I=>n,UU5>5UPU>v,!=invi,Uv==-IiO+PSU!>,!5>i+vP!i-vCiC51CC5=OUPi5+CP-U-v+IPPiI!vPU5CvCIIO,C>nOO+0OO>+PW,n=QP-_+vvI,>nnC>nIO,=>UP>=Z=>U++I=5PCiP>n>!5IOnOIIIv>-{!->>n!5+!v>U=5IP!I,vi==5&O=t!UUCO!CUnv!Po5!N+,v5==Ii5n{bC-=+IL5i5v,ICi>CU,in+vCIfO>P5InO>=*UU>CU-U++nP!5+!!IPnUCO-,=,=MUC>=PIU5v5,,OCv>IUni=+,COH0>55>n1>UmG=P-iiv+i>n!C-,nOPIP-I>P-,nn=PIUi>!OP!55CU,5C!=,-!=P7n5+CCI5>n>7Pi5-viiY5v=>I+>!=-U->.,>U-v,PI5n+>,vn5=-IUO>=P-=>O,iU5>+cZin!nP5iDCiIInI,>-PC-f>n>>+IPUOv5Pn5CvPIPnv!5IPOw^!UUO5!Oi-=P!+OUvPUC+,!II-+5=v-sOO!In!+,I-i5vvP!n!v>Inn>=vi5OPLRUP+I-IUnviPn>v!PI!+KC5iPnC,O5=vx!!nI+nP+iUvZ-v5-=,IIOnCv-v>5!-UU+>?vi=vO--i=Cv,CO,CO-i>PP=-nC,r=O>=M-!5P/-,OnvCi-!+C=UUI>-!O-C+CPnOI+!!O>iC!U>ni,,55vnfO5v+PP-U=vn-UiCC+,vOPCU-U>!wC-=+IJUi5v,IC5!CUPvnO!iI!+>=in,C5,nUOCvPP5-+=,n>UvCI+nv PIU>U!!UC>=PIUUv5,,OCv5IUni=+I,Oxf>55>P,+U5SI-nOOv>-Cn,=U,hOOPi-nv>DCi,>iPiiC=v!O5PC&IPOI,IIn>i9nnv=PP!O9>i-I5Iy5Uc>{CaIPnOIi-vv#!IOi+nPCi-CP,U5UP{I-+n=>n!CU,i-=+v!C5!=+,,nICII5+-=UI5OvI+UP>I!n>5+CPPiUv=i=n!C=5lvU,,5-v=!,>P>n!oiC+rin5PCO,On_==Ii=U65UU.i!C-==>-C>UPnPLni=-IiC}=vU>>+P!U-v-P3O>+?iU5I=nIOO==!U,>CIOU,+,!>>UvPU=>5C+,in==CI=vOQ>-v+PIUU3+5P=5+v+5,nn!+I=>I=5U5>P!(i!vIUIinCi,5+vPP-!v/C=-!OUj+UIF>!PU=+i,P>iC5,+5l=nnnO5=.Ui+I!I>>v!-I5iC+PTnYCO-5OnL+n+>!!=U61--O5n!5,i5nCC5vO,=--O=nf=U!+UU-i>+v,Pv!Ci,nnC,v-,O-BOOn>5IPOndOPIOSC!,,n5!+Ivn=pI55O+!I-f>O!5ii!6Pnn!,!Iinn=C5PO+AIU>+5!5>=+5-5>1v,,_n>P!I>O,)OUi>i-Ci,=i!IUv#O,I5+=,IEnSIi-=vWI4OPD-ICi5+m,inICI5>nPPIn-CR,iO-+PjCU*vU!(5Uv>,iv5Cv5vO-=vUI+!!!>5+i-!i>PI,,5>=PI=n=IU-vv=Ii-O=v-5+,vPPOn!CC,CC-=>iCCU=nn+=iUPi!=>!5+UC,P=n!=iI!Oi=+-5=nJ5OC+nPi+5v=-:+nv+ivv5I-I+>,!!U,j+!-iv+nP,5!v/5n5H,=--Oiw+O>+PIIi5:+U!+vC,UUvU=I5nv==PnICCI,i,_I!5>C!vUvizP!IvCn=P-OOU=d-C>v-Ci+!iUiiO!vi-+UP!5nn+,COO+I!PiirCP>iivn,=5nC=IInyI!-5=U!!U=)w!vii+vi+5n=iI!O>C=-U>IMUO=+-I5i>jjUUin!+iUCPC=-vOi!!5+=+I=UP=+-5+PK>,+5C=,nPn>:,I=On8iUUb=!Ui,+>,UvUC+,CO,,nnU+!!P->+j,P>5vnPi5+P>,OOUP+,=v=;{n&=,:UOi^I-n>=v!iivPCi,=CI*PnvCv=}O,=>I+Oi7=PP+-P3IOn5=vnI>!=v-=+-%=i-+OPU+ivCi+ni=-nUO,=v-,wPE=iv+i,!i-v+,O5+,-I>+C9!nUC+==OP=+U5i-C,Pvni!P5Pv-C5-UCUI5n!+!PiiOv>-U5OCIi+vU!=-Pv,a5n>O=I!iP+IPn+5v!,n5-v=,vn+I-I+Ons!U+c+PPiIvnU=++=--P>,!-iPCCM=-v+PU!U#v+UPi-!iIUvUC-5nv==Pn,CnI,OvV5!vi-!O,5v,P,,Uvn,!5PvvI--5=nUUie+C,,+OCi,,n-=>I-O>=s-+=v!I>P+vP>++vn,,5n,5I->,=vUiO>!PUR+PU>5!MI,i++PP,-v5,PnCO>!nU,+vI5>5k>!C5&!UiC+i=5IOO=IC-i>=g>U-+,PP+>vPP=5i=PnPO5=O-==--P5-vi,-U=v>iIn-C,I5Ci=UUPC5=>n+++-+U>f=U-i5CnUCv5,,I5nfIv-Iw,!nU>+.UU>C!v,O5UC5ICn5=C-,O=IgUI^-!xiC!=,U>i!k,,v-=+n+O,;OU5>OU,U+vI!ziOv5,iv1C+nUO+=CU,FP!nn>+=U,>5PI,O>vPvI>va,UInCI!-n>=6!n>O!nUni+!viIC-CC-UOP=+-O>n-Oi5!,U,iU!ni!+P!v5-n5,nOU>:jCi,hOPii,v-,>5-C>,zn+,v-I=P4vU>d+!ni,+ni55-=,,vOiC>-PO1dPO>+!IIii<+UPi-!5iPvCC>-nO,_v55=5I>-C=n-O>C/iPn>>P+-PvUPvn5v=2,n,Ox!kU++C,IiCCI,nn-,U5<OPIiO!>W!Pii>n-Ui5eOPC+=PO5+>P,5nIC=!CO!=vUIO>v+PCn,,P,>O,C=InOiTUO=>U!,U>vUiU5+vCI,vn,UIv+,P5U=vO-5in+iP++>vOIU++v=iwOm,Ln,OUIiOI+>-=iU!iiP5iv=5IOP,v5vnLI,n>C+IiO=+PU-+<CO,5nv,I-!nv==U-O=!-UO+UUiiC!+,in-,UI,nv=,OPO=!vUiv!!-i+vOP+v-C>UCO!,U5+n=IPn+85!-5,+v,i>PPPi-55PO-PC5,!UP>I!n>5+!Pni-+=Pv5+,-,+nn=!-+=+!PUI+n-=>+v>P,O!=v-vvC==Iv>P-!-Q++-PU-ai,U+UvOin+=CP5POC,>nv=57vU-)OP5+,!,PU+nP!iP+v,-I5CO-UUz>CP,>OviP,5-C>,-n>C:I+CvzIOP>v!>>++nP,inP5,-O,Cv-in>8P-L>P->i!=IPi>+!PP-+5PP5Cn>Fn-,>v,5O5=>tCOnv5UC>ivnU>55CO5>CUI5UCC4I5>>v-UO>tv,P-+P=P-5O>d+5iO>!-OvyiIm5,jI,n++POi+v=P5IO>CI,iPCv-C5I;U-C+O!Cin5CxP5i>U,,-i+nIviv=5-ji,v9,>+!=PIIOnI5-!>nj--=>v!++-++Pn5!C+5+OP=I-nC=I+UO>P,O5sCnUC5=vvIPC!C9-+CP=-n-+U-UUOdn-=iP!I,vnOPv55nv=-OPOC-5UC>RP->>!,iP5=v>,vO,Cv-,O5uIO->n-Oi-v,iI5>R+i-55Pn-!=!=5-=>vQ=>5+!Pni-+=Pv5+,-I!C>#!-,>5-iUCCYPI>5cvin5=!P5Pns,-5>nC,tn+CCI=O#+nUO+n!nP++v=-n-nCVU-PO+lOUnrOP5+,!,PU+nP!iP+v,-I5Cn-UU:>CP,>OviP,5-C>,-n>C:I+CvWIOP>v!>>++nP,inP5,-O,Cv-in>BP-W>P->i!=IPi>+!PP-+5PP5Cn>%n-,>v,5O5=>6CO+vnUC>iC5,On=,CIiO==>-->,!P>>+P!=iiCP5Pn5COI=C-IPiiv-_!-+vnUI5-v,,5viCU-Pv5C>5>>+I+n=>PP!Onk&Pi+5P,,55h,--,CC,C-!=II+nv=5-Bi,!-5!n>CnICC-wPICOK!U-w+U!>ii!5P=+vC5IUCi=IICOI-,-S+C!55P+UPv5>vv5Un+!=-PCi,vI;=,Iv>n+U,IiCC5U,v,PU,nO5PrnnCP0Ui5=+P+Oi!iP5nO!CICv+,nIO>PIOUi>,!-i>+-P>i:v+ivvi=-5CC5Ei-n>C-!ii}>!O>M!>iP55C}55nnD>5D>=I>i!7-!I>O!-U^+>PIin+O=O-B>,!I5=O+ -OP+U-Ui+CCiPOP!=-Pv,b5n>O=I!iP+IPn+5v!,n5-v=,vn+I-I+OnT!U+r+PPiIvnU=++v>P9>!0+U5CCb=-v+PU!U/v+UPi-!iIUvUCO5nv==PnP>vIiOvx5!vi-!O,5v,P,,Uvn,!5PvvI--5=>UUi&+C,,+OCi,,n-=>I-O>=J-+=v!I>P+vP>++vn,,5n,5I->,=vUiO>!PU3+PU>5!EI,i++PP,-v5,PnCO>!nU,+vI5>5u>!C>xP,U+nvC=-I=,=+UIOy?OU5+iU?iivIP+ni,iIvn=:InO=i!lU>+>I!ii!n,O55Cv5+n>Ni5vnuI!i!_!!->IZO!v>v!>U=v5,,I5nhIv-nV,!nU>+6UUUvvnI-+>vCi=O=,=nPO-Iinv=>PU>>vOP+n!PiPu5nPUIJO56=U+>+U,in=+P!+iCnivnO,P5vvU=OUPOC)5UU+-UC5I!+i!++Pi5i+Bj!-,>5-i-w+5!IUC++P>vICOI-vv,5--O!k,Un>,!nUC+OU>+-vv,-n+=nInCa=v5nOU==U>+,PO5iviiCnI!ii=nP,-5=v^TOUP+nPUiU!vP+>5C-I,CI=n5+n2=+i>>O,U5n+,I5+vvI,>nnC>nIO,Pv-==PKUUPC!!Nn+C=PnOCP>InOI,>-OOC(PU>J>!OOI!-Us5vv=IIC,=--,=Ibnn+>=,v5+vPPIi=v--5v-C+5iO5,>iO>n{,U,>vIP5i=>i,5,COI5nOI,-nv+0=OI=OIOi>+=P,5+P+IP+-=55vCP,,UI>U!>>O+i,-5UCnUI++v=5Pv>Pv--+,A>in+OPv>OfVP,+U!*I=+O,PI-C5,*n,>i!=U>+-P,5PP>,5+U=iInOCIIn9=O-PO>TUU!+!v ,PniPv5nCI^!IvO=!--=+-!OiU!iPC+nC>i=n5,-I5C>=ZO!+I-OiP+O,!5CvC5-n+!(IvO>I+nICI,*U+!,!O5!v=,!vOC+IOC>=O5-v=IU>P>n!biC+<in5>CnivvIPII==U=vUI+P!I>v+UPCiCCII,5>,OI+OOI>-OC,I+>i>CP-5,v-iCn-C-5=n5=yI+OUoIU,f+P!Uv+=,-i=C-,OnU,iI+Cn:>n=>O-5i,v!iP5!P5P=nU=IIUC==C-U>v!OUO!!PiOO!OP=+nP!I*C!==nPCnI+Oi_U-I5,!IU=5U!diivO,!n!O=IPnn=+--OO<v-UiOCPPC55CUI-CC2Ini>O--OI=-PUiO+C,iviCU,CO-cP-P=no=nP=5Pi>5vUUn>NP,U+5x=5IInC=+->=+!!>-LvP>>}C!i5>C!OIvCIDi-->O-nO,a=!>i,!pU>>>C+,CO,Pi-5OOz=n,+I!Ui>==,,iCCUI-nUP!-POIVn5v>=x=>n+PPOiU+}PC5v,UI+CI,Unn=!!,O>=O-viP!,PU+nCPi=O!,55}>PIOn5=+!!>P+-U55!P!,tnP=ii>n+=C-,>n!!>5>=!+iIv=UIn-Cn,vOUIU-vOv-i-:+5!IUC++P>vICO5PvI,i5=>!Inn5=>J^>!+IUiie!v,=vUPCIEC5,UnOO=IVU,.U!=>=vCPSn-!nIOn+G!nF>URO-C+iUiinvii55iHoU+vI,=IUOvx>-vLUe=ii+iPv5+vI5,n-=,nIv+P+-n=AMiUC++!C+ivCPCv5C!Inn-C=IvO+--n==-I=OOh-U-+!!ii++vPn5IC!,nnC=,I>O-!UP+iCC,Uin>CUICOv=C5n>Ok+i!=-PiiiP!POnPvC,5nU=-nC>II+nCD!-ni>!,UP+-vOi>5C,!IOCiwnnN=5!O>P=:UIin!OPv+8Cn5nO5=O-=C,wIUU>>P!in!jPi5Iv+Ii++=v-!O-HCOC+-!->=+5PWi+vU,In,,+-PCO,+n==i!n>!=_U,i5!nP++=C55-Oi,CnU>5ITnChP!i>5+>UC5iPiIUn5=v5!>PfIUnq5!CU v-iI5OvC,Pn>,>IvO>I+n!CI!!>5>=PU5IvUi=n!C=5jvUC55IO;-i-C+-P,i-!CPv5-C+Innn,rI=vn=vn!>>!,iOviPi+Cvii5n>PCI!v,=nnCC=-,n++C!C+5v!,n5-v=,vn+I--OOUb5UC>5!Ci,+=Uf+OCP,OO!=CIC=-QPn=>i--inH>PvO9C,i>5^PvI5Ci=Cnn>I-POO+-UIi>!U,!+=P5i>v3Pn-,=PI!OiCj!Ii++5PPizv=i;nC,IIiC>IIOP+P-UiiVvUU+Iv5i55nP=55noPC-=>!!UnO+>!v5P1U,i5>v=I5C5=iI=>U!,U,ZO!nO,+i->5vvv5i5H=5IInC=+->FI!nU-+iPviivv,P5CP=5nO!=n-K>v/v>I+OIvi)!5Pv+OC-5In>,U-!C=IiUP&!!5>,+CU++i4d,OvnP>5Cvi=>U,O=znUi+UU=5-sO,>5v=PiUOi=n-CCP!,U-+OUni=v,Pi5rP_IPn8I!5-vntUOv>I!>in+>iI5>v>5-5C=UIPn+=O-nlP8>iv+-,!5IvU,>>=CxIIn5p!O!>,!!>P=OrOOUvPiniWCiI-ni,j--Oi==O++PI-O>;iUi>-!!5In>C>n-Oi&-OU>-=CnO=>UviIv>,n5>,II>n>I-IC>U2P-+>O!n+P+=U25iPP,>n>=,5COiIi5.+!!,i5!iP#i>C,IPn,!CI=O!/U5O>+_+>U>=Pii,+vP>5O,,,>OICg-P>-}!U=>iP-i=+o,P+OvPP>viC,-CCvN(U>+nIiOPvO-5>-C>i!+v!N-ivv,n5+=5!vn5+O!Ui5vCP55CC,,=vB=CnPO-I5-v=C!n>I+vUv55CU5inI,vIIO>Kn->rImvi-+!!>inv55!ni=+IIOe9,-iO>!IU>+nUOin!7Pi5CC+,CCiC<-5OI=C-+>>UIiO!OP85Iv5I!C!=i-+nCcp5>>!-CiI+UP>+OvZ,Pnivn55Oi=n-C=v!UU!+nP5in)I,-5iC+U1O!=-In>P-PUn>nU,U+vI!jiOv5,ivJC5-!nv==UPOC!+UIvPP+ivv=iii=v55I5b4>nO>v!5iU=II=5iL-UPn5!CiO+vaI5OCU,nO-+OI-ii+,P-5>v-,>5FC+5vO>,=-I=-/5O>+UU!iO!O,-n,,II!CO=!-5>UT5>!>OPPUC+5PU5-PCIIOn=!-vO^{I-5+!!5iU!iPU+vvI,>nnC>nInv}--!O>onU5!!Pi+ivv,!5-CC5COIHnIv>v,5-+6>P!i,v5ii5vv=II5U,--IOUQ>OO><!Pii!UP+iCC,5Pn5COI=CCwI-U>>-OUl+PPi>v!-UC+nPI5I';vePrRghzQkGaBUYtqvpbuKzzCmKxzZwS={"dkl7ek,.lGM,.jn,*)*lnjHdl,M*jj**e,n.,*d),nd4HkH7H7H*GMM.k7l3H*.nGG*,llM,","l,k.ndn7n3eH7*n),)*dGj47k477k)7n),e,","*4je4kd*ed.4)ej*MnMnHk3,*Mj,4jkenHn4eGkk*eHG*nM","Hk*djGj*HHn)ln,3e3G4HkMHj7)77,,jH4dnMd.G,,n3kGdde3l)d*37deH*eH,M*Hd..MG,3H7","elM,lM37HdljH.43l7n7d74Mke774M3..d4ed,4MG3)j4n.k7*Gl4kklnG4Gee.4nn4n,.keM.7jdn*","3k4GMjn),)374)k3H,kM**e3kM7kj*.3GGM47G34ej.klM,lnk.,","7ee7,)jlj47GH*ln,,kk.)MHkej.7k.4e4j.n43Me*7e4lnjkMG*dk44*.k.7jd*3*edHe*Mn4ek3je","MHeen,3.HHHl3G34HGGl)kl,l.,)nG74*kn,Hd3lld*,7G*djj,*.G*4dndnk,","3ded)le7GGj*G.,Ml*3Md4lHnljH4.l7dM3Mk74HG7knkjjH)j.7)lljeGel*3*n7k3*)l,.n,3*jj","n))nM.l34eGG3ddMHHnek)e,Hje3.*j),7,*M3jnGM,)e","*)k*H*H)l.4d**4,H.3d7HH,,,7))437M3j43*H)jM,kG.3.lleMHMG3lGM,4.Gnkd7Hj43,k)4jddk,3..MM*d7k4MdlGkd)nH*dM**)4kG*l...j*)n)j)G","7334l4)j7*de,*7,e**4j)*37l7M3j,H)jeMjklkH4leGM4.l*MHek4G.Gjk4H,.kH4jl*dd73Me,*kGM33d,d)3l7.3M.jlG,ddkM47,,Gek*7*.4.j).e.7,dllH3.,","jMjGH*)47nH)n,Me**HdjMdlMn3jen.Med*M4M,MjG*7l.l)l74l)jHljeM3)4HeG.jd)7,4*j*.,MGj4),)4jGH.3.MedMd7)M*k.7.jk.n3M,d*M.d33l,.7HG)3,4d,jG.**dle3M4,..kM.Med3ddlMM744Hjj73kneG*l3kn,k3HkHG,.,el*43e*3,73le4H3),dM7)n7.lnj73n,d43.l,lG*d3,Hd4,,k4M.nj**)**,Hj47l73.e**.jlGjnn**jHj7M4d3.ldGl7H7*l.k37.ed4k,,n,4M)j),,e**7*k4,dndjj*.3ej*)Me47nMMlk4n3nejkj**dH.e3GG,kMj*j),G*3,)4MGe337.74))4j*)7jknjdH73d*kMe7j*d77)G.nG*nlM,*M43,.M)3d7nn4G4*.)e,n3nk)lMknM,e3Hj),4d**nGG)MjMne3de3*j.**7*j4jk*3de*M4e,lj.Mel)ej33.Gk,ld*k.nM*l,7neGedln)d),4H..,*,ee44n*4,MeG,,*.Mn43MMn4.n3Hlke7MHMd3*)e,*.Ml,G7**,kd3kMje*GneG,3.4ed,MMe3GddMjG4k.Mj,3MG.nl,M,.ede.lG*7jl,jdj4*,*H3jMHMM*k3G.,GG3kk)7H*M)3n3en7,M.*Gj4H*le)3nGkd,*,eGjk,dGM33M3ejj4jMMHjG77nM,*,jj*3)HMe*n*eMHM)43Hdkn.3,en,),HM)M,jl33eHHHej,lMdkM),HGj4H,kjej)kkjG4)H3M,Hj3j.*j4H.k773,H,j.4)3,,.eHMdHjHM.))e34M*73d,l4M*d4*nnHeG)en*G,G.,jdkjM4jnndeG,n*,dMkl)H)G,,k.k.Mk..l,*G*)nGjlM,..)*dH,dlk.*M*3jMn733*Ml.nj,l,nddn)4H7l3)7lkk),4*)HG)k*ejMM3..,He.)jGM)*4G7)e),k.)M*j7,nkdjG4,n*G7.lH*ljH*,Gee)4je*7HG*lGee3e,l3),k3,3Mln7)MG.3GGjM4edj))3MHG)Mjje))7eMel,jddd,7HH)j**4n)G)74*nnd*4j),G)jlj7dM,jj3.,j4j4d3dd33e,klnMGjl,d,M4d,djl4MMe)H*l7,,HHHn,*jGH,k,nk.7dHddddlGlMe,)dM7nM3j.l)jkj,jjM3*3*4MMl.,ldGMjk)4.)lHG7l4Mlkd*)H*,,Gej377nH,HMGkjnM)kG,**e)),Mj**j4MkndjGj4dde7edMMenH*lkl.H4,3H34j*l*M37Gj4Hl4e.e*3ed3jj.3,4k7j3HGjG)ldH.je7jMl4Gk*47*ekMlMej7j,dMdG)jdken,G..eM4n.Hj3*7.3,Gk7ej734MG3ln)3M47*kj3l,7Gd.l3,G.7)nG3jd*74n*j33GG*jGkekd,7nk.GeH*.jdl)H3Hjkndn,G,*kjHj,,d,k4MMj4Ge)j.43jl.)n,G7jlkjMMjlH*.nk3Mj3H3l73,Me)nj)j,lk)G,n4H3jl4nM4H7j4H*73*.Mj7e4.ndeH*H)3*e3,ll*MM*G3ljnl,dH,**eMMM7lGMj.GMjnk)n.*,jjkj733,*d33)*)k,H.3,nejl4Hlee,.lGd*jenl7GGj,kelMH7,3l)*Mn7d34Hj*G3dMG)d47jMjk4,7GG44),GM*nnn3el3dHk,k)jl7.M*H7*Mn7Mkdn,,Hk43dGGe.e3k3n,3ejeG),nHk7dn4.,G3.kMMk*47d*7knnne7GjdM4GH*4e*H)3**l.37,4,k3kMj7j,477elH*jd*,j7G7e.MG)k*)e7H*lne7*e3de,H3H,dj.G,djnen7j3Gd.7e47.jMl**,G3jGnj74l3,33.)Hj3jd3nlH7H,k*nGejl,eGHMH,G4Gk.4e4)*n,.*.)G.GjMe3*nM7jdlee),d3MHkk).M*43Gj*,l3Hj*j.l.kkG4Mn),G73,HMd43M434d.)jkk)*7HH*,d3l4e*Mekn4Menll*lM.,H4jk4j*l*d7M4dG.*4M.kH*MGkl7en.7j)nd)Mke7eHjk4,)M3,.j*l.jGjl4Gk,3ekjMj,k.jMn*e33ekH.k*3kdG,llH)43lHGe)M.)jHkeMMGk7Gj4G.3djGMH7MM)HMG*MH4l7*,.n43*jeHG4*,M,l.*7*H*j.)n,l)474.MMlHn*G,H).n,ejH)elHH7.*Hn)dHk4*ne7el,*G7k,.7G43.MG*eljkeHl.nd*MGd)jkkM*Hjj*)Hj34)3d*Gn7HnG,n*)jej*,d7HdjMl3G7dn,e3,7eMjnMldjG*7H4jn.7)j*,)M*jG.kj44jle.,e*HHnlHH*lGMn.n3j)l,7.*7n)ee,M44GjG,lkkGM)n*ejndk7,eeH.j3*,jGlj,Ml*.jHj,ek,),)jn)HeH,7HM,e3H7n*4G,M*.7jdne3H**Mde,.nkHnj*d7eHHj,e*.*77Mjj4*eej7k3jd3k37HjjnH*GHj.kMnGG34**,Hk7enMlHMldd,ek,jelj.G7)G,*7.djen3M4Hded*GnekH*47,Mk4n,7j7..**4GjHl44HknHeG7)M7l74j7MkdM)3.nGedld3Ge3M4,Hd4dd,jln,7M3MG,*n4l4.)Gj,n,G4G.4GMkMk),j7)nddM,dj3G437e)MjG)4,7lM)jkMn,d3*M4HM7.),MGj37.H))He.4jk.*e34.*.nM4M*4e,d.Gj*,3HjM3MlM)),M,e)3)kljndj3M7,ljH*GlkHMlen,j).lddkej,Ge.*M)l4344,*d3kG.e)4keHM,*njnG3j*n7*,*j,dGlMl*GMHk.e4kllkeeM)))7)7*llG,de3*4M7nn*jjG,G*MeM.H*ejdl,3nGd3*44ej.M.Ml4M*,Md47Hjn,*.,G4MeleH43,7k)74Me*dk)djd,3kk)d3jj7nl.MH)*e.*,,*,34j.),kM34Ml37,n3,H3MnHj7M)l7k3.dj77ld3*77He,H,Hl,n3jMnklG3,733,)e4lMn*eMG7)4.l*7n)*n4kj7).jGjGMe),H,H*d,dj*,H33jM**lH4nn373Gn.Mj.3kedk)njl)k)k7d,GeHd3G)lH),eGe,nH*dn33GH,Hl7j3l)3).H7,j.7j.lk)n*)37,kjj*7elH)ll3,.4n7j*Md3lk4dkG,H4k7ke)3k3njjj.eM)d*)n3Mk.,kjdnHk)HM)).e,4nHlnd73le7e,Mj4*n)4)e*7le.Hj.kkek3j3jM.d4jkHl)j.l*klllHed)jleeM)4eGde*4d7,lG,e3G*HMMjekjjHj7)en3HkjeMkjdddj,,n*,eHlje*,Hl*dGn**43)k.3HdM7*4j.,3d,M)nd.4ekk7,dn7Hd3k7H)7,Hnj3Hn*H.3,H,**j).e*4jk4j4.7)nke7n,,*He)*kdld*77*deM,lj3HHnle7,.74443MMGMn7d4djH)4l*.*l7jnM4.Gdnd7lHe4G**4M3,MdMeHn)4nM)l)7nM,d*,j,Med7d73HMkd*7lGM,e7,4d7l7HMl)G3Gk*dlej4H)nj,MkGeG3,.k))dM4,7HGj),kj4*3ej*43)7kj,.*Gen*7,kH3jeG7lH.7,4jdkl.,e74,n3edM7e4k*jl.njkMe7l74)jdeMH,.d3)ldMG.M*ndkj*,)jMnnje)GG.)4MHn*lM*74kdkMM47l,)j7lnne*3*4n7*MHd73j.H,3n.dnl*n33ne7,j.GMldk,H,d)e,.,3Hj4l)e",""};return(function(a,...)local l;local o;local d;local s;local f;local h;local e=24915;local n=0;local t={};while n<188 do n=n+1;while n<0xeb and e%0x8a0<0x450 do n=n+1 e=(e*28)%12885 local p=n+e if(e%0x1612)>=0xb09 then e=(e-0x18d)%0x2460 while n<0x2a5 and e%0x3cce<0x1e67 do n=n+1 e=(e+847)%30200 local f=n+e if(e%0x9d6)>=0x4eb then e=(e*0x1a7)%0x6c9d local e=77685 if not t[e]then t[e]=0x1 d=(not d)and _ENV or d;end elseif e%2~=0 then e=(e*0x165)%0x3246 local e=42559 if not t[e]then t[e]=0x1 s=tonumber;end else e=(e-0x181)%0x9e0f n=n+1 local e=47294 if not t[e]then t[e]=0x1 d=getfenv and getfenv();end end end elseif e%2~=0 then e=(e+0x2b5)%0x8350 while n<0x101 and e%0x1398<0x9cc do n=n+1 e=(e+783)%25238 local h=n+e if(e%0x4cee)>=0x2677 then e=(e+0x359)%0x8927 local e=95890 if not t[e]then t[e]=0x1 end elseif e%2~=0 then e=(e+0x48)%0xa014 local e=58062 if not t[e]then t[e]=0x1 f=function(t)local e=0x01 local function n(n)e=e+n return t:sub(e-n,e-0x01)end while true do local t=n(0x01)if(t=="\5")then break end local e=l.byte(n(0x01))local e=n(e)if t=="\2"then e=o.BVKaMnXp(e)elseif t=="\3"then e=e~="\0"elseif t=="\6"then d[e]=function(n,e)return a(8,nil,a,e,n)end elseif t=="\4"then e=d[e]elseif t=="\0"then e=d[e][n(l.byte(n(0x01)))];end local n=n(0x08)o[n]=e end end end else e=(e+0xe0)%0xa407 n=n+1 local e=42394 if not t[e]then t[e]=0x1 end end end else e=(e-0x3c1)%0x8747 n=n+1 while n<0x167 and e%0x2514<0x128a do n=n+1 e=(e-654)%20121 local d=n+e if(e%0x1b26)>0xd93 then e=(e*0x3ca)%0xb070 local e=72784 if not t[e]then t[e]=0x1 l=string;end elseif e%2~=0 then e=(e+0xde)%0x8feb local e=90003 if not t[e]then t[e]=0x1 o={};end else e=(e-0x29a)%0x2a4c n=n+1 local e=61929 if not t[e]then t[e]=0x1 h="\4\8\116\111\110\117\109\98\101\114\66\86\75\97\77\110\88\112\0\6\115\116\114\105\110\103\4\99\104\97\114\107\75\122\106\103\98\79\122\0\6\115\116\114\105\110\103\3\115\117\98\79\80\117\81\97\66\69\121\0\6\115\116\114\105\110\103\4\98\121\116\101\121\122\101\106\65\102\88\120\0\5\116\97\98\108\101\6\99\111\110\99\97\116\120\72\121\65\70\69\74\74\0\5\116\97\98\108\101\6\105\110\115\101\114\116\120\75\97\72\84\100\103\71\5";end end end end end e=(e-597)%23939 end f(h);local e={};for n=0x0,0xff do local t=o.kKzjgbOz(n);e[n]=t;e[t]=n;end local function p(n)return e[n];end local r=(function(h,l)local a,t=0x01,0x10 local n={{},{},{}}local d=-0x01 local e=0x01 local f=h while true do n[0x03][o.OPuQaBEy(l,e,(function()e=a+e return e-0x01 end)())]=(function()d=d+0x01 return d end)()if d==(0x0f)then d=""t=0x000 break end end local d=#l while e<d+0x01 do n[0x02][t]=o.OPuQaBEy(l,e,(function()e=a+e return e-0x01 end)())t=t+0x01 if t%0x02==0x00 then t=0x00 o.xKaHTdgG(n[0x01],(p((((n[0x03][n[0x02][0x00]]or 0x00)*0x10)+(n[0x03][n[0x02][0x01]]or 0x00)+f)%0x100)));f=h+f;end end return o.xHyAFEJJ(n[0x01])end);f(r(209,"5n_7o!FqDG-WL(U37nF7W33a__!nq(L33L_n!_GU(-UG_q3__ZqUL_n!oUq!-7L3d(7(3q_!Do(L}FoWF_GoWL(U7G3L_LDLWDUF7-qUW!(og-7-oqDn!3D3ULn-oWWFLGnD_-!(qFW-(UL_3o!_D?Wn(UnAq-G_Wn;G_ooLFWLqD7W7nno!q!Un(L7o!3-hL733_7UFnFq7-o(Gn-oqGDWW3G7_oWDqoGqG(Gi-7D-LWo33<WoDDL->UD-L(L7LF3GUUD_GFqGH-n(n3F!qv3onGU(n3-_(!W-n(F3-!!DGW!UGUP(V CFnG{LQ_G_-F_DWU(H!n(!W77F7Ln3!nUF*--W(<D_WFGD-WCGFLF_7!oDo(G3ooLq(GnUObc7q3G_WDqWFU!nDoGDGW!n7oq!!-!LGU3W-U-oDq!-FUooGoo-nL3(!_G7_n(o(--(DgD!DDGWq(UnLo7F3LFDOWnZ(7WFW-nUnn_!3GnW!(W3WoWn_oo-nL3:n_LF_-DULn3o(qDWGW(U((7/7FSGFLFnU7!DLWnWGnq_Gq77FFFL-3__qF>WnUW3q!XFo-!(GGGLW_F!WDFWDUq7_DnW!Ln8__U!GG(!-D-U3nUooWp(!U(_-q!G-LWU_W(U(o(Dn-L}G_UFLG!WDQn77o:_loU-3U(!_oWDq-GU_n-o!_!!7W7Tq77F!DGW(3LoV!X_-!DW-lU_UD!DFU-bD7^73_3!(Lw_!_3DUW!LU3G_Fqn7oF_LFdFF_GFLnU(*foFoG7GFGLq3G_GqLWGUoV_!{!}-DLoGLLW_-!qLl(!_!o(FFWq(W3o(T33!3D(L33(!oq(WG(WM7_LXo77G_Ls_}!-FqWzU3nL7(q_7DFqLq3ooDFU-7LGnno(!_GnFLG(3-_q!-DL(-a77LDD-D(_EFn_3U_3DWWGU-n-!3WoLXk!noqqDDL!GeW3_no(DWUUnn_q!7-oL(3!Lo37!nD!LFfL_(!!q(W!B3n73D_qDqWF3(_oDo-!Wq^nn3F__L!WLEUq_GDUGn(LU7_!!nq-FED33!_nqnW7UWn37G!nqD-oGoL7_F!!G7W3nF!rq3DFLGU(LD3q!DDo(73-oU!WGF(L((nU3L_WGpL<nq!G!F-L(qlLoF!n7w!3W(3yoGDUWhLq337G!-DGFoG_3q7!DW-mU3__7WDeD_FGGD3G_oqoGWUF3DoDqoG3WWG(LL_(!-WU3o3-7LDoD(LnV((nh;FoG7Bo_LF7DG-(L7aoo(g!7!G_Ln37oFFU-(Lqn__GF_DGFDGq3(_DqqWoLF<^_Gq!G!(!G3"));f(r(174,"t=RwCjOdVHp-xus9jRppOH-xexjjpx9dsOHV9-CjHp-9wpH=xCw=O9u9nC=FOwRKOj-j=ROC-s9DjC-O9Owddp9Ou-w=ssCRV-sRwpdu-ARddVxC{-jHH9pV22-9=RORp-WRCuwu9dwupRswwpHCspR6juuw7pVOux-C4pxjn-d*-OpjjspV9-Ompp9jCjHw9xC-H=sswjRrdpwwdduVRwOuxsUudR-=:COXp-=RjdHOsCC9Hssuw-RxVRC8VuswwOVOx9ROd=Hu9wjcp=9Cw-HY9lsxCwoCCwHO9Rwuwi9=wxH5uwRdV=p9RnVOuV%ROOHH=V9xjz==jx-,9wCdp=uojFHwu9CdOxuHwR=xO9RxV0xHRAdVxR=wCd-H9RCdpOsHspws9uC=HpsjwjHV-p==dHuVRsj=-w9H7Ojx=djuxR#V9dppARjdpH9swRd-xxCXVxsp=zHbdw"));local e=(-o.WUKMjwWf+(function()local d,n=o.Eiskjgvw,o.ZUkSVOBn;(function(t,e,d,n)t(n(e,t and t,e and n,n),e(n,n,e,e),e(e,e and d,e and n,t),e(d,e,t,t and e))end)(function(f,e,l,t)if d>o.UjrbijHS then return t end d=d+o.ZUkSVOBn n=(n+o.nRrUYCXM)%o.qZJs_XHu if(n%o.uAjfxlyn)>=o.KIk_nxbo then return e(l(l,l,t,e),f(e,e,f and e,f),t(f,t,e,t),l(f,e,e,t and f))else return l end return e end,function(t,e,l,f)if d>o.zSoLZoTm then return t end d=d+o.ZUkSVOBn n=(n*o.SetnPQGy)%o.KQFrwtyA if(n%o.pBKExaJY)>o.uAjfxlyn then return l else return f(f(t,l,t,t),t(e,l,t,e),l(l,f and t,e,t),t(e,t,e and f,f))end return f(e(f,t,e,l and l),t(e,f,l,l and e),t(e,e,e,f),t(e,f,e,e))end,function(t,e,l,f)if d>o.D_C_SbTK then return t end d=d+o.ZUkSVOBn n=(n*o.ZFoqVvgo)%o.IFuzYzof if(n%o.AwCqpGlF)>o.RTSKZROw then n=(n-o.IKilaXIj)%o.OdhgcuJK return e(t(e,e,t,e and e),e(e,e,l,t),t(t,e,e,t),f(t and l,e,f,e))else return t end return f end,function(e,f,t,l)if d>o.bJrhApSn then return e end d=d+o.ZUkSVOBn n=(n+o.AcnKuZlq)%o.RrwTfc_t if(n%o.mcLZvlnV)>o.cYhbVpsQ then return e(e(l,f,l,t),e(t,f,l,l),e(t and e,e,e,f),t(e,t,f,f and t))else return t end return f end)return n;end)())local p=o.HvgawJel or o.GfzzRdEr;local fe=(getfenv)or(function()return _ENV end);local _=o.ZUkSVOBn;local f=o.BxBOBLUQ;local d=o.WTWLQavW;local h=o.YJrDyhaA;local function ne(u,...)local y=r(e,"PfaoyN&^p0)9e;5t^aNy^5)p;otea&N=^))yp5y5&ay;^&)Ve)tyf5yp9p;ee0tyf)yy&5^W5059e;yopo0y955pNt)9;Nt5T;y0^f0^9NwBapy0^p09;f;Sf9y &)0NepeNaaye^yatNtpR));y&&0eofea.fay&ypo0)^9)&eet&aUNNe5e55ftNa5y&^?0)t&yte;9e&ae;y5fefyNfei5Nae);0a&Q5;0^995&}eo&&xeN^Aei&NooNep&9M;)dya5p&ea)tyo1Ha)Ny);;&_<a9Ny^5)0fQyoN0pN9a;9fpt0N9po9e;9g)o)NNpe9aetFyoN&Np0)oe9.Da)fN^5)9e^tya;NppM)f;rto5Ny0^;)o;y5;ayNo^p00e5tpfoy&&o)De)tafoo&^o0e9^0e;&ypa)Ny^5)p;aCoa&N3&)59tto9oppf));NkeaNN)&))^e;waae&oNe)o;)t^t;N)py)ee;t=a;N&^f^N&fp)9y;5spoa;eepti;9-ya5Nppa)e;&tb&0N^pa)p;atepyo&p9feettpaaye^&)Ie)tyf5y9^t0ee&tka;9N9paf5ypptoNoe^e9po0a59o;&&0+9)5ym5op^5ee905Nk)oyN5tpayyN^y)yN;0f)5;p_atfpe&&))omf);aa)t)^p)pff))Nf^e)aeet&ye9f)eeoe0tafey&^/0)eya9Npyo&e0&ec5)fyo5p5;a9;5&f!o)&yp5opa5yeo0&^p)9y;5&t0^;i5efNyp^)atNppa)e;&n%a)py95)0;atea&N_^))y5aopaoye^&)se)tyf5N,9a0;e&tYf)yy&50p5aaef0yy&)0y95N0tyty400o955y}5opeNtyf^Naw9oyN5pp9a;eC&yp0)pN)5;pWaaeN&p_9)fy(faeNa^e)&a9&&totN^0)aeet&a*y)^ye&7ptofey&^r0)ey55o)pa&tN^ea5)fyy9^a0te&5tNfN);yt^f9N&pf)peN))apot^o0^e))&f&ye^o09^pt9f9y9^ay)e95ok)oNN;0op)p^)555fpya&e0&e+5)aIo5&p0a9e5&fmo)&yp59p5a e^N&50peaf;-poaNep&9#;)Kya5Nppa)e;&mef^o5^5)p;atea&N.;N)9;etpaaye^&)Fe)tyf5yppk)oe&tWf)yy&50t;h5ef&yu&)0y955pfaoe^y0^f^;ftpop&ape9&5x&Noy9epp905).&o,N)py)5;pyfpp^opd));yt5&^0f;p5N;{t)ayy5^p)a;)t;ajy);at99N;efey&^>0)ft55^&ya&etyeitfafye)^0aeyaNyt^0)oe;t^afy9^N0te09;;5y^ym&0p;j5;)xypeNppa?0eo5)a)Ny^5Voo8tea&)5^))yaetpaaye;y)5;ftyf5yp^a0ee&tUf)yy&50pe0t)f&05&)0y955p^.oe&&0O9)yft9aa&apeff5qM)0aN5pp9ao)i&o39ppy9;;p=aae&N9e));yt5apNap95a;Wt)ayp)^p)aeeNyaly)^y05a&tafe)oN;pyey55^oya&e0&f55)fyo5e&0a9etNfOo)&yp5t^af*eo&&vp)9yf;y^oaNep&9#f&1ypeNppa)eoy}*a)Ny^5IN9t5&a&N#^)tte5tpaa))^&)L;0o8a;yp^a)05atpf)0o^^0pea5ef5y+900;;etNfaoe)a0tt&5y&eop&apefyo;t^f5N5pp9a;e&fo_9ppy)5;pf;oyN&pA));yt5app5p9)&;5k^oay59y9feet&a<y)^y05;fs;fey&^Dtpeyy9qNae&e0&ei5)fy00&ptM9eyyye^0&yp5ew59heo&05p)9e;5Tpoa&Np&55;){yo&&fpa)efN1ta)9a^5)pat50f7N7^))ye5tpaaNopo)5B0Naf5py^a0ee&tpN)Nf^e;^e0t0f;^t^p)feNtffaoe&&0t9)yay;y&&t0o)oe)4)oyN5pp9a;ew&oUN)py9N5NGt&pN&p/))fy!No&0fpo)p;^o0oaNN9p)aeet&atN^pf)N9N;efey&^20)ey55fpya&e0&e{5)fyo5)y0a9e5&f1o))ie5eN50f9o&&Op)9y;5f^fty&p&9b;)(ya5Nppa)e;&_la)Ny^5)p;aoe&aN ^))ye5ttatNo^&)Me)tya;ytNtp&e&tqf)yy&50pea5ef&y}&)0y955pfaoe)&ee9)5y>5op&ape9&5%#)yo&N^N0e;eY&o_N)py)5;phaaeN&pU));yt5apNa^e)&;^QfofNep&9Z;0t;a^o^N505eptafey&^}0)ey55fpya&e0&e25)fy&&)05 /%o0f/&t9yp559a;NN^)y;^y)fep59f&oe&&0-eNpf0Nt;&N0y9&5WZ)5&0&t0ajf0^a0o55pN)5;p4aaeN&p}5;fyifa;Na^e)&No)y0pe))e)N;&t&a3y)5dT;a0N&pt9)5aAooaNNf9y^&e0&e=N)^^0t;yt59t50f#o)&yxfa;yy4;o&&Tp)9y;5kp^^0ep^9{;)jya5Nppatuf&#fa)Ny^5)p;a;e0N0r^;)pe5tpaa&Nt;)p&&tNf5yp^a0ee&tVyypy&t0pea5ef&y!&);tt559ftoe&&0by5^)05p9505t^t0NoyN;Nbo;at;;m&oQN)py)5;pya&eN0p&));yt551^a0oae&oyfa^NN^p)aee9e55at50tN&59ffty;^ 0)ey^)ytNitN))^)a9f^ya&p0a9e;5o)9&o5pt9p5aBeo&&G2)tNf5:9o&Nep&9/mo0p0&;&pN9L;&61a)eo55pye;tto^N+^))yoayfpNeNeo5tooy^0N9f;5?tC5N)pNp&e^09e&5ef&y1;&qoftyefNyN&&0E9)et)G)&^^)^e98yt&f5&f0y9a;eD&pN)at9fyNf^^)o)&totfo^&0a9N&^e)&;Yye)p^^_8)oeet&aDy)^y055eoaftN&^A0)ey&ppt0)99})a&Nf&pp^5f;eM0NNpa0e9f0f955a_eo&;ot5a&y^^&00N;p&9{;)*ya5Nppa5e;0ata)Ny^5_fyf^f0p^L5)f%at&epp995p3;5)N;0f&p5^s)a^Nap&)0p5seof>e^99pett^ooNe^^Ne;)tpaoN^a5)9eNt5ftNd^5&a&f)p9a;e{&)&9t;yaca)&00y;s;ffNf9Nt&))^;Nf)yf^70y9V5;5;y&^_&o9^9tf&yaye0o00^o))ey55fppt0Ne)aNfppapNee;)a^Nao))y09ef5y5Nf^o&&kp)NfaN0ootp^ey;^yey55;Nypo)e;&Wla)Nyt5tIfattatNC^))y9ptfyNetNao;yy0)5o&;^o0ee&tZ^Nyy&50peo5ef&y<^e)o955pfay+&&0z9)t&aaop&apet)5/G)oyN50;9a;;u&o6N)py)5;pf^aeNpp8));yt5apNa0<)&;ot)ayy5^p)a;5t)ary5^y5oeptafeNp);0)e055a&ya&e0&;af9fyyy&p9t9e5&fsye^^p59t5afpo&&7p)9pt)Ipo^Ne009Z;9nyyLN9pa9N;&yfa)Ny^5epNNteatN:po)y59tpaaNa^&))e)tef5yp^a0e;&tBaoyy^W0pey5ef&Np&)0&95t!fayN&&0l955yf^op&ype955uV)ytN5p)9a5N%&opN)ppff;pv^ae&fpm)9;yfu&NNapf)&;et)ayy5p);oeeteas&a^y05epEyaay&^^0);g55fpya^M)peg5tfy&a&p0o9e4&f&o)&;p5ed5afpo&p>0o9y5p.po&Ne9h9 ;)I&a5&ypa9y;&qja)Ny^t)p;ttea5N=^))ye5cpaaN&^&);e)t&f5ypp;0ee0t!aoyy^p0p5a3Nf&y0&)0p95fafaoe&p0keN5yf&op&ape9&fNS)y4N50O9a;eB&oEeypy9^;pI5aeNpp7e)5;t5oaNapy)&;et)ayyt^p)Neetta>Ny^y)aa9taafy&p^0)eN55f5Nt&e09eY59fyot&p0&9e5&)No)&yp5905a/eo&&8ft9y;5OpoaNep&9U;)00a5Nppa)t;&k a)Nyfo)p;ateapN(^))y5BM^aaNa^&9^e)tyf5y5^;0eeetTy)yy&t0pea5ef)N0&)0y95fpfao;&&)a9;5yfyop^0pe9&532)9yN5p59a;;h&oaN)p)9&;p_paep^pW)9;yttapN&0N)&;zt)NNy5^0)a;5t0a Nh^y;5eptafey&)N0)e955f;ya&e0&;aN9fyy&&p0t9e5&fxye^;p5ef5afpo&&Jp)e&5pnpoeNe)_9>;)?yyrNtpa9^;&oea)Ny^59)5&teoaN+0q)ye5tpaa&p^&)^e)+Mf5y)^a0e;pt=aayy^o0pea5eap9f&)0;95f&faoe&&)a955yfpop)ype9&5hfeotN50o9at&u&orN)0&9e;p/5aep0pr));yf*o;Nap0)&;0t)ayy5^p9oeet5a_N^^y)Keptaa^y&^00)e055fpya^5x^eqtyfyy&&p0a9etpa0o)&tp5;a5aEeo&^a)o9y5)rpyeNep&9S5evNa5&Npa95;&1la)&&0Q)p5bteN;Nq^))ye5fNaaNN^&)5e)t&f5ypp00e;EtRftyy&50p;ya0f&y9&)0)955pfay5)o0se&5ya;op&apeepf;/)yfN5)f9a;eQ&ya)5py9e;py)aeN&pq9e5ft5o^Na)p)&;.t)ay&9^p)eeeiNa!ye^y05;5taa^y&^o0)ey55fpN9&e)aeb5tfyo5&p)yet5&f;o)&^p59p5af5y5&T0p9yt^{poaNe0p;0;)foa5^fpa)e;&fao;Nyp5)p;0tea&N3pe)5e5:0aa&{^&)je)tyo}yp^50e;^t}feyy&50)eat0f&yN&)0y95t)a5oe^y0T;&5yF5op^y)f9&5tJ)NaN5pp9a55atoH&)pye^;pIaae&p0^))5Nt5ytNa^e)&5a>5ay&l^p;yeet&ary)^)05;Ntaa5y&^a0)eyt&fpNh&e09eg5)fyNSpN0ae95&aoo)&yp5e)ty?ey&&,0)9y;5Qpyy00p&ef;)f;a5Nppa95fo%CoeNyey)p;ateopp;^)9^e5apaaye^&)c;;tyaeyppN0eeptWf)N^&5)oeat)f&yf&))&t95pf5oep00,9)5yn50&&a009&5N+)oyN50)90;efyoB0ppy)5;pfyoyN&pt))5tt5apNap59);kM)ay&^^p)aee:payy)pN05e;tafey&pa9^eyk*fpy;&e0&eh5)f0o5^N0ae55&fao)&y)e9p5tmeoe&u0^9ytMy;oa&)p&95;)qya5&)0p)e5NwxoyNy^5)p;afNa&&.^)))e5ttaaNapy)!;)ty0fyp^o0ee^tFf5N;&50peaNtf&yf&)0p;^5pfaoetN0-995ya.^N&a0e9&5eb)oyN5pp5A;ef^og&Npy)5;pfyo0N&0a))5}t5apNap_9f;H2)ay;e^p)oeet&o5y)pp05;otaaty&^meNey4afpyp&e))ewtma;o5^e0ay05&ffo)&yp59et9Veo&&Of^9y;tdpoa^)p&ey;)J)a5Ntpa)e&w?Wo5Ny0a)p;)tea&&^^)90e5=oaaNt^&)F;0tyooyp^p0e;9tzf)No&5)eea55f&ya&)0ye65pa&oe^)0.eo5y%5Na&a)69&tyJ)oyN5)5eN;ef)oL5^py)t;pfeaeN)00));yt59NNa^;)&;Aatay&a^p)teet&aVy)e505;;taapy&^R0)ey2afpN&&e0pe859fyNYpY0a;A5&y;o)&yp59p5y?ey)&Kp99y;t(poa&ap&9U;)uNa5Nppa)e&e.=o5Ny0a)p;ateapey^)90e5tpaaye^&)(oftyooyp^o0ee^tqf5)^&5)pea&of&yf&)9y5 5paIoe^y0je&5y-5&;&a0e9&5ND)oyN50)ao;ef^oi0opy)5;pfyo^N&0a))t&t5apNap5eo;hm;ay&e^p)aee!pa&y)pp05t;tafey&pa9aeysofp&;&e0&eC5)a9o5^p0a;f5&fao)&y)p9p50zey&&Cp)9y;5&&oa&ep&9N;)Uya5&)0o)e5^(k&4Ny^5)p5yfpa&&a^)5oe5tpaaN5p9)E;;tyo;yp^a0e;pt0f)Np&5)^ea5ef&Na^p0y;o5pa&oe&&0B9)5NZ5yp&a)f9&5a8)oy0opp9);ef&ozN)py)5t&gaoeN&p&));9t5ap;4^e9&;Z()ayN&^p)p;at&oSy)fp05e0taf;y&^y9oey55fp5&&e0^e>55a;o5&p0a^55&ffo)^&e99pta>eya&Vp)9y;5o&oa&;p&99;)Zya5&)0y)e5pUso&Ny^5)p;&-ea&&%^)&Ne5t0aayete)D;5tyf5yp^a0e;ptof)N0&5efea5ef&y&p^0y;y5pfpoe&^0v9t5yX5e;&ape9&5&2)oyN5ppy^;eL&oj&fpy)5;pCaefN&pn));et5apNa^eo9;lt)ayN0^p)aeet&9Ny)^y05;9tafey&^/apeynofpNp&e0&e65)f)o5^e0a9e5&ffo)py))9pt&4ey)& 0&9y5ypeoa^zp&&&;)qNa5N0pa9H5tria)Nyoy)p;otea)&0^))ye59^aay;^&)1tttyoayp^t0ee&t_f)05&5);eatpf&yg&)0y;95pa&oe&p0i995yajot&a)/9&f;G)oyN5pp9N;ef)oxN9py)t;pYao0N&p6));Nt5apNa^eoe;Zj5ay&a^p)aeetp0yy)p005e)tafey&^1=feyBofpyo&e0^eh5)a&o5^90a9e5&f*o)&ptf9ptaWe;5&Qp99yt5a)oa&)p&95;)fla5^5t))e5yTT5tNy^t)p5>tea)&0^))ye59;aay;^&)y5otyf5ypy40ee^tdf)^0&5)peat0f&yx&))&?y5paooe)00H9)5y>5o5&a0e9&5aZ)oNN50)eo;ef&oF0^py)5;p7aotN&0v));Nt5a0Na^e)^;8t)ayyt^p)aeet&a)y)py05;ptafey&^a_5eyttfpy&&e0&e45)^9o5^00a9;5&ffo)py))9ptaneyf&iee9y;5aToa&op&9&;)fya5&)0t)e5&g<o&Ny^5)p;a#9a&&f^))te5tpaaN5pa)j;etya)yp^a0ee)z)f)Ny&5^)ea5;f&yJpp0y;a5pf;oe^00D9)Nyb5ye&a0a9&tyg)o)&^ppe&;e5^onN9py99;p%&yNN&pR))9Nt5a0Na^e;9;Ml5ayN9^p)aeet&^)y)p005;ytafey&^.)pey?afpyy&e0^eFteo(o5^e0at05&fko)&y0&9pt&#eo^&>p99y;5faoaNep&9f;)4ya5Npfp)e5)1mo5Ny^5)p;y^(a&&N^))9e5tpaaye;;)O;ttyftyp^o0ee)Nof)Ny&5&eea5;f&y&p^0ye55pf^oe&^0G9e5y}5e;&ape9&5&#)oyN5ppy^;e &oZ&ypy)5;p*ao&N&0f));^t5apNap5);;B.eayNt^p)aeet&afy)p^05e;tafey&pa0eeyrafp&a&e0&edtef&o5^;0a;f5&fKo)^&0e9ptpheNf&Zp)9ytLC;oa^op&5y;).ya5&)Of)e55jr^fNy^5)p;ahya&&o^)9ee5t)aayep0)*;5tyaoyp^a0e;p{;f)N0&5;mea5ef&Na^^0y;y5poooe&&0mee5td5yt&a0e9&5di)y&^Nppe);eo^osN)pyeKfNkayNN&)&));yt5ap&N^e9);<foayNJ^p)a;pt&ofy)pp05e0taa5po^E)eeyt)fpya&e0&555)a^o5^00a9e5&aayt&y)a9p55/eo&&:p5;f;5f)oa&0p^9f;)f&&9Np0p)e5aMRa)Ny^5;&;afoa&N9^))ye5d)o5yep5)M;&tyf5yp^&)pe&>&f)N_&t00ea5ey9y+py0ye55pfaoe&&et9)tt45yo&a)N9&5Mf0oy^)ppeo;ef9ohN)0o)5tanaypN&pa));)i9ap&e^e)9;ft9ayyt^p)&5Nt&aVy)^00te0taadNt^,0)eyt0f0yo&e0&tN5)a5o5^N0a9e5&fH0y&y)09p55(eo&&gp)9);5aaoaN5p&9f;)f&opNp0e)efoDZa)Ny^5ea;af&a&Nf^))Ne5tpo5ye^&)Ce9tyf5yp^&tte&K&f)ya&500eatH^0y?^e0y9p5pfooe^p0o9)t&W5&B&ape9&5&ptoy^Tppeo;;R^o.N)py9atfvaaeN&0f)9;Nt5ae&9^e)&;:=eaNyt^p9yett&oay))t05eptafe^9^-);eyJ.fpya&e0&yf5)apo5^)0a9e5&fJo5&y)o9p5^deo&&O0eep;5f5oaphp&9O;)f&oaNp00)e8N-Ca)Ny0uea;afya&&#^))ye58)a5yept)O5ttyf5yppy);e&1)f)y&&50pea5e&pyG^t0y;p5pfyoe&&e)9)tpZ5o)&ap59&5wc5oy^app9o;e(pou&e0o)55;vaa;N&pk))5&f)ap&p^e5*;Lt)ayy5^))a5vt&o&y)^y05ept)feN0^g0)eytzfpyy5b0&;y5)f^o5&p0a9ef&f8y;&y)a9p5aWeo&&op)e^;5gpoaNep&9J;ecya5Nppo)e;&rxa)Ne^5)p;at;a&NT^))Ne5tpaaye^&)c9)ooaoyp^a0e5&N&f)yN&50pea5;f&yc^^0y9t5pfooe&^019)tqJ5o0&ape9&5(*)oy&ypp9o;eE^o_N9py)55ckaaeN&pf));yt5apNa^e)&;R*aayy5^p)oeet&acy)^y05opoafty&^k0)eytff9ya&e0&ooNt^&ot&p0a9e5&fVo)&ye5905a1eo&&Mp)9ye5e^ooNep&9h;)+ya5Nppa);;&C!a)Ny^5)peao9a^Nl^))ye5tpaa^e9&5ae)tyf5yp590ee9t:f)yy&50pea&)f&yN&)00955pfaya&;0j9t5yfoop&ope9^5++5y;N5pp9a5fs&ofN)ppe^;p#aaeN5pm)9;yt5)fNapf)&;%t)ayy5^peneet9a_yt^y05eptaNoy&^N0)e&55fpya&erteb5tfyot&p0a9e5&ooo)&Np59;5a(;o&^a0p9y5oxpopNep&9M5ef9a5N5pa9o;&SLa)&&09)p;0tea9Nb^))ye5UaaaNy^&)fe)tyf5yp^;0ee5tCaayy^a0p;y13f&y0&)0)955pfay5)o0*ey5yfUop&ape9&a^%)otN5p;9a;5k&N&;apy90;pf^aeN^pG9K;ySayfNa^e)&5Nt)aNy5^e99eet&a>&&^y0tepta0Ny&^e0)e&55fpyape))eJt^fyot&p0N9e5&a+o)^op5955aGeo&&r0)9y553popNep&9{;)f&a5&Npa9e;&%aa)Nypa)p;5teoaN4p^)y55fNaaNN^&)ee)tef5&55)0e;ftDo0yy&t0pe05ef)N0&)0y95O^fao;&&0y;o5yU5oppppe9^5K_));N50y9a;5M&o?N))y)t;p:taeN^pW);;yt5oyNap9)&;&t)ayy5^p9^eeQ&acN8^y05eptaoay&^;0);y55f)ya&e)MeRt&fyy)&p0t9e5&ayo)&;p5ef5afpo&p&ja9y507pN9Nep^9i;5<yoa^fpa)e;&a0a)NN^5)e59tea&Nn9o)yettpya&5^&)^e)t9f5yt^a0eoetxa&yy^f0pea5ef&y^&)):955tfayp&&97;^5yf)op&5pe9;5n-)y&N50N9a;tl&oRN))y9t;pflaeN0pm);;yt5yfNape)&;et)ayy5^p)tee%^aCN&^y05epta&5y&^50);N55f)ya&et9est^fyy9&p)(9e5&f)o)^fp5ef5af0o&&_p59y5eUpoeNep&9!;)fya5&^pa)t;&,ja)^yp9)p5ateooNO^9)ye5jyaaN9^&)te)uyf5yp^00e;?t(apyy^;0pep&of&y&&);a9550fayy&&0y;o5y(5op)lpe9^5jL));N50y9a;eh&oqN)pyfG;pI5aeN&p:9y;yt5NtNap&)&;at)ayy50peyeet9a8yt^y)Nepfao5y&^&0)e)55ftya^H)teO5)fyyt&p0o9e5&o)o)&yp5905a7eo&&Np)9y;5LpooNep&9u;9Aya5Nppa)e;&tb&0N0^5)p;afepeNE^9)ye5tpaoye^&9^e)tNf5y0^a0;e&tKaNyy&t0pea5ef&yQ&))a9550fao;&&0f9)5yjtop&ape9^5_w)oyN5pp9a;eO&ooN)py)5;9ffaeN&psaof9^&00)ae^dpaaNVpp9N;ffaay&taoN)^y05epoppf));NdeaNN)op90e;*05CNa^09?e;0N9e5&fko)&;p59p5aoa)5&Hp)9y5fipooNep09z;5f;a5Nppa)t;&ifa)Np0^)p;ateaeNZ^9)y5Vt9aay;^&)fe)tyf5yp)00ee^tjfeyy&50pe&7Nf&y/&)0e9550fay5)o0x995yfBop&ape9&f^i)oNN5p)9a;es&obpypy)5;pQoaeN&pg)9;yt5apNa^;)&;(t)aNy5^p)aeet&aFo)9o)oeptafe&&;&0)eN55fpya&;0&eu/ofyot&p0o9e5^fAo)&)p5905ameo&&Sp)9y5a8pooNep^96;9_ya5^apa)e;&gfa)Ny^5)p;atea&N:pa)ye5tpaNNf^&)Re)&ypf)0;5vpe0t)f)yy&5t^af&SpN9o;5c&a^N&^0o;&&0m9)5yn5opNae9905&<)oyN5t9a&y5&t05efp^9);p#aaee05Nf5yf&o0e9otg:^oa&&p0Nf^;)aeet&0y)9;&:)a;tNaCy&^j0)op&^pp)^&;0&eT5)fyo5&p0ateoefFo)&y)#2^5a.5o&&fp)9y;5gpyeNep09<;)*ya5Nppae0;&Yya)NN^5)p;ateatNT^;)y;ftpa&ye^&9;e)tpf5yp^a0ee&txa)yy^a0pe&5ef&yK&))o955;faoe&&0.9)5y&Gop&ype995kGeoy&yle9a;5q&o9N)pN)5;0>aoT&tpg));yC0apNo^e))50t)ayy5pN)ae;t&aI0o^y)feptofey&^V0)t955f9ya&50&eQ5)a&yf&p0N9e5;f(o)&yp55;5af>o&&fp)9y;5Ep0tNep09U;e1yo_Np)pay;&Soa)&^^5)0;aIFa&Ny0o)ye5tpoNye^^)le5%;f5yp^aeee&tff)yy5^0peN5ef&yB&)0y95&&faot&&0o9)5N65o5&5pe905waooyNtpp9o;eO)y0N)py)5tfcaa;N&pqto;y:fapNa^e)&;{t)N9y5^9)ae5t&avy)^yf^eptNfey&^/0)ey55o&ya&t0&eo5)fNo5&p;99e5)fEo)&yp59p5apNo&&Np)9y;5%poa&a)o9?5n4yo*Nppo)e;^6ga)50^5)p;a,%a&N8^))yogtpa^ye^^)Z;utyocy9^a)fe&tNf)yy&50pyp5efeyJ&)0y955pfaea&&0^9)5yW5o0&a0a;o5TfaoyNtpp9o;eD^oVN)f0)5;p3aomN&pQ))5yfoapN0^e)p;R*aayy5^5)a;ft&a^y)^&05;)t&feye^q09ey55fpya^;0&e^5)fyo5&p0a9e5pf yf&y0N9p5aReo&^0p)9e;5w;oaNep&9wtNVyoyNpp))e;^Dna)e;^5)5;atea&ND^))yoWtpapye^^)re;tyf5^t^a)Ce&taf)yy&50p;y5ef0yw&)0y955payot&&0o9)59.5op&ape595,J5oyNtpp9a;ed&0;N)p^)5;)Saa5N&)&Eo;yVfap)^^e)^;UT,ayNa0f)aeet&^Ny)^N05ep&9fey0^l0)ey55fp&a%y0&ey5)fNo5&90a9et)fMo;&y0f9p5&JeN&^xp)9p;5X0oa&yp&9!5y3yofNppN)e;)Wva)&p^5)e;atea&NO^))y;.tpa^ye^&)}e)tyo5Nt^a)ae&tff)y^&50p5Z5ef9yq&t0yey5poaNp&&0&9)5N-5yq&a0a;o5LffoyN5pp9o;e}^o3N)f0)5;p!aolN&p*));y&zapN^^e)e;OAfay&u^9)a;ft&aNy)^y05ep^pfeye^/0)ey55fpyata0&e^5)fyo5&00aea+of:ya&y0f9p5o?eo^&Wp)y0;5Vpoa&2p&9d;)fyyoNpp0)e;pnRoaNy^590;aufa&N^^))&e5x)ooye^e)Me9tyf5yp^a90e&t^f)yy&50pea5ef)y-^f0yeN5pfaoe&&)99)5em5o;&ape9&5?a&oy&ypp9);en^oAN)t;)5;51aaeN&p8))tyy)apN0^e)^;#t;ayy5pf)a;at&a&y)^9055prefey;^+09eyt&fpyapp0&e&5)f9o5&90a9e5tfco5&y0l9p5aFeo&&yp)9^;5WpoaNep&9h y#ya5Nppo)e;&R3aeNp^5)p;atea&NV^))pe5tpaaye9t)Le9tyftyp^a0ee&.zf)yN&50pea5ef&yE0y0y955pfooe&&0F9)85s5op&ap;9&5*W)oNN5pp9a;e+&oRN)py9f;p.aaeN&e0));yt5a0Na^e)&;jl)ayy5^p)aeet&aky)0505eptaf;y&^W0)ey55fpya&e0&eT5)fyya&p0a9e5&N0o)&Np5905aKeo&&C0)9y;tOpoaNep&9J;)a5a5Nppa);;&gCa)Ny)p)p;atea^N ^))yettpaaye^&)Ve)tyf5y9^a0ee&tH&oyy&50peo5ef&y=&))y955pfaoe&&0x9)5yopop&ape9^5(d)oyN5pp9a;eT&o N)py)55NqaaeN&pA9^;yt5apNa^e)&;jt)a;y5^0)ae;t&a{y)^y)eeptafey&^709ey55f0ya&;0&ea5)fyo5&p)e9e5&fbo)&ypt9p5a+5o&&fp)9y;5JpoaNe0&9?;ehyofNppa)e;&^0a)N^^5)e;atea&NK)&)y;mtpayye^0)be)nyf5y0^a0;e&taf)yy^y0pea5ef^y:&)0y95tefaoe&&0E9)5yk5op^ope9&5_G9oyN5pp9a;eW&o+N)pN)5;pJaa;N&pr));yt5apya99)9;st)ay&5;5)ae;t&axy)^N05ept;fey^^z09ey5tfpyapa0&ef5)fyo5&p0a9e?of3o9&ypt9p5ozeo&&op)9y;5,0oaNep&96;)rya5Np");local n=o.Eiskjgvw;o.tTfvdSDn(function()n=n+o.ZUkSVOBn end)local function e(e,t)if t then return n end;n=e+n;end local t,n,r=a(o.Eiskjgvw,a,e,y,o.yzejAfXx);local function l()local n,t=o.yzejAfXx(y,e(o.ZUkSVOBn,o.BxBOBLUQ),e(o.kFvnXTRn,o.GtyurVmA)+o.WTWLQavW);e(o.WTWLQavW);return(t*o.ZhdSrClY)+n;end;local ee=true;local k=o.Eiskjgvw local function b()local e=n();local n=n();local f=o.ZUkSVOBn;local d=(t(n,o.ZUkSVOBn,o.jPqjTvkU)*(o.WTWLQavW^o.EDlfsf_Y))+e;local e=t(n,o.bmKeusek,o.VMSJcpFV);local n=((-o.ZUkSVOBn)^t(n,o.EDlfsf_Y));if(e==o.Eiskjgvw)then if(d==k)then return n*o.Eiskjgvw;else e=o.ZUkSVOBn;f=o.Eiskjgvw;end;elseif(e==o.VPZwhchE)then return(d==o.Eiskjgvw)and(n*(o.ZUkSVOBn/o.Eiskjgvw))or(n*(o.Eiskjgvw/o.Eiskjgvw));end;return o.igczwkvk(n,e-o.IoLbBvXf)*(f+(d/(o.WTWLQavW^o.AuiPMoib)));end;local j=n;local function c(n)local t;if(not n)then n=j();if(n==o.Eiskjgvw)then return'';end;end;t=o.OPuQaBEy(y,e(o.ZUkSVOBn,o.BxBOBLUQ),e(o.kFvnXTRn,o.GtyurVmA)+n-o.ZUkSVOBn);e(n)local e=""for n=(o.ZUkSVOBn+k),#t do e=e..o.OPuQaBEy(t,n,n)end return e;end;local j=#o.AEPOPEHV(s('\49.\48'))~=o.ZUkSVOBn local e=n;local function le(...)return{...},o.DI_lpAQE('#',...)end local function te()local k={};local e={};local s={};local y={k,s,nil,e};local e=n()local a={}for d=o.ZUkSVOBn,e do local t=r();local e;if(t==o.WTWLQavW)then e=(r()~=#{});elseif(t==o.ZUkSVOBn)then local n=b();if j and o.ptXCgLvz(o.AEPOPEHV(n),'.(\48+)$')then n=o.TKJlfrFt(n);end e=n;elseif(t==o.BxBOBLUQ)then e=c();end;a[d]=e;end;for y=o.ZUkSVOBn,n()do local e=r();if(t(e,o.ZUkSVOBn,o.ZUkSVOBn)==o.Eiskjgvw)then local p=t(e,o.WTWLQavW,o.BxBOBLUQ);local r=t(e,o.YJrDyhaA,o.GtyurVmA);local e={l(),l(),nil,nil};if(p==o.Eiskjgvw)then e[f]=l();e[h]=l();elseif(p==#{o.ZUkSVOBn})then e[f]=n();elseif(p==u[o.WTWLQavW])then e[f]=n()-(o.WTWLQavW^o.H_QJCLwL)elseif(p==u[o.BxBOBLUQ])then e[f]=n()-(o.WTWLQavW^o.H_QJCLwL)e[h]=l();end;if(t(r,o.ZUkSVOBn,o.ZUkSVOBn)==o.ZUkSVOBn)then e[d]=a[e[d]]end if(t(r,o.WTWLQavW,o.WTWLQavW)==o.ZUkSVOBn)then e[f]=a[e[f]]end if(t(r,o.BxBOBLUQ,o.BxBOBLUQ)==o.ZUkSVOBn)then e[h]=a[e[h]]end k[y]=e;end end;y[o.BxBOBLUQ]=r();for e=o.ZUkSVOBn,n()do s[e-(#{o.ZUkSVOBn})]=te();end;return y;end;local function de(t,n,e)local d=n;local d=e;return s(o.ptXCgLvz(o.ptXCgLvz(({o.tTfvdSDn(t)})[o.WTWLQavW],n),e))end local function c(g,y,s)local function de(...)local l,j,b,te,u,n,r,ne,m,z,k,t;local e=o.Eiskjgvw;while-o.ZUkSVOBn<e do if o.BxBOBLUQ>e then if o.ZUkSVOBn>e then l=a(o.GtyurVmA,o.KCtfIdSg,o.ZUkSVOBn,o.CyMiATQK,g);j=a(o.GtyurVmA,o.WjsxcLDA,o.WTWLQavW,o.EBjuuMaV,g);else if o.ZUkSVOBn<e then n=-o.QInLXoAS;r=-o.ZUkSVOBn;else b=a(o.GtyurVmA,o.H_QJCLwL,o.BxBOBLUQ,o.atBiekvR,g);u=le te=o.Eiskjgvw;end end else if o.YJrDyhaA>=e then if e==o.YJrDyhaA then z=o.DI_lpAQE('#',...)-o.ZUkSVOBn;k={};else ne={};m={...};end else if o.WTWLQavW<e then for n=o._uhPYdVV,o.EBjuuMaV do if o.kFvnXTRn~=e then e=-o.WTWLQavW;break;end;t=a(o.FmSswawJ);break;end;else e=-o.WTWLQavW;end end end e=e+o.ZUkSVOBn;end;for e=o.Eiskjgvw,z do if(e>=b)then ne[e-b]=m[e+o.ZUkSVOBn];else t[e]=m[e+o.ZUkSVOBn];end;end;local e=z-b+o.ZUkSVOBn local e;local a;function UHwfyuwPsGck()ee=false;end;local function b(...)while true do end end while ee do if n<-o.QIeHifbO then n=n+o.gnEXqKPm end e=l[n];a=e[_];if o.yrzqfM_y<=a then if o.XQaLhPZR>=a then if o.UcFQKhYj<a then if 105>a then if 99>=a then if 97<a then if a~=94 then repeat if a~=99 then t[e[d]][t[e[f]]]=t[e[h]];break;end;local d=e[d];local a=t[d+2];local l=t[d]+a;t[d]=l;if(a>0)then if(l<=t[d+1])then n=e[f];t[d+3]=l;end elseif(l>=t[d+1])then n=e[f];t[d+3]=l;end until true;else local d=e[d];local a=t[d+2];local l=t[d]+a;t[d]=l;if(a>0)then if(l<=t[d+1])then n=e[f];t[d+3]=l;end elseif(l>=t[d+1])then n=e[f];t[d+3]=l;end end else if 92~=a then for n=17,56 do if a~=97 then t[e[d]]=y[e[f]];break;end;t[e[d]]=#t[e[f]];break;end;else t[e[d]]=#t[e[f]];end end else if 101<a then if a>102 then if a~=103 then local e=e[d]t[e]=t[e]()else t[e[d]]();end else t[e[d]]=t[e[f]][e[h]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;end else if 101==a then local p,o;for a=0,6 do if 3>a then if a<=0 then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];else if-2<=a then repeat if a>1 then t[e[d]]=t[e[f]];n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]]+t[e[h]];n=n+1;e=l[n];until true;else t[e[d]]=t[e[f]]+t[e[h]];n=n+1;e=l[n];end end else if a<5 then if 1~=a then repeat if a>3 then t[e[d]]=t[e[f]]%e[h];n=n+1;e=l[n];break;end;t[e[d]]=y[e[f]];n=n+1;e=l[n];until true;else t[e[d]]=t[e[f]]%e[h];n=n+1;e=l[n];end else if a>2 then for r=47,97 do if 6~=a then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;p=e[f];o=t[p]for e=p+1,e[h]do o=o..t[e];end;t[e[d]]=o;break;end;else p=e[f];o=t[p]for e=p+1,e[h]do o=o..t[e];end;t[e[d]]=o;end end end end else t[e[d]]=t[e[f]]%e[h];end end end else if 109<a then if a<112 then if 108<a then for o=21,72 do if 111~=a then t[e[d]]=t[e[f]]-t[e[h]];break;end;local o;for a=0,6 do if a<=2 then if a<=0 then t(e[d],e[f]);n=n+1;e=l[n];else if 1~=a then o=e[d]t[o]=t[o](p(t,o+1,e[f]))n=n+1;e=l[n];else t(e[d],e[f]);n=n+1;e=l[n];end end else if a>=5 then if a>=4 then repeat if 5<a then t(e[d],e[f]);break;end;t(e[d],e[f]);n=n+1;e=l[n];until true;else t(e[d],e[f]);n=n+1;e=l[n];end else if 0~=a then repeat if 3~=a then t(e[d],e[f]);n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]];n=n+1;e=l[n];until true;else t(e[d],e[f]);n=n+1;e=l[n];end end end end break;end;else t[e[d]]=t[e[f]]-t[e[h]];end else if 113<=a then if a~=113 then local n=e[d];do return t[n](p(t,n+1,e[f]))end;else local a,y,r,s,a,a,k,o,u,j,c,b,h;for a=0,6 do if 3<=a then if a>=5 then if 5<a then a=0;while a>-1 do if a>3 then if a>=6 then if a~=7 then t[h]=b;else a=-2;end else if 1<=a then for e=20,53 do if a~=4 then h=o[u];break;end;b=c[o[j]];break;end;else h=o[u];end end else if a<=1 then if-4~=a then repeat if 0<a then u=d;break;end;o=e;until true;else o=e;end else if a>0 then repeat if a<3 then j=f;break;end;c=t;until true;else j=f;end end end a=a+1 end else k=e[d]t[k]=t[k](p(t,k+1,e[f]))n=n+1;e=l[n];end else if a~=0 then repeat if a>3 then a=0;while a>-1 do if 3<=a then if 5<=a then if 3<=a then repeat if 5<a then a=-2;break;end;t(h,s);until true;else t(h,s);end else if 3==a then s=o[r];else h=o[y];end end else if a>0 then if a>=0 then for e=48,77 do if a>1 then r=f;break;end;y=d;break;end;else y=d;end else o=e;end end a=a+1 end n=n+1;e=l[n];break;end;a=0;while a>-1 do if 2>=a then if a<1 then o=e;else if 2==a then r=f;else y=d;end end else if 5>a then if-1~=a then for e=30,68 do if a~=4 then s=o[r];break;end;h=o[y];break;end;else s=o[r];end else if 5~=a then a=-2;else t(h,s);end end end a=a+1 end n=n+1;e=l[n];until true;else a=0;while a>-1 do if 3<=a then if 5<=a then if 3<=a then repeat if 5<a then a=-2;break;end;t(h,s);until true;else t(h,s);end else if 3==a then s=o[r];else h=o[y];end end else if a>0 then if a>=0 then for e=48,77 do if a>1 then r=f;break;end;y=d;break;end;else y=d;end else o=e;end end a=a+1 end n=n+1;e=l[n];end end else if 1>a then a=0;while a>-1 do if 3<=a then if 4<a then if 5<a then a=-2;else t(h,s);end else if a~=4 then s=o[r];else h=o[y];end end else if 0>=a then o=e;else if-2<a then repeat if 2~=a then y=d;break;end;r=f;until true;else r=f;end end end a=a+1 end n=n+1;e=l[n];else if 2~=a then a=0;while a>-1 do if a>2 then if 4>=a then if 4==a then h=o[y];else s=o[r];end else if 6==a then a=-2;else t(h,s);end end else if a>0 then if-1~=a then repeat if a~=2 then y=d;break;end;r=f;until true;else y=d;end else o=e;end end a=a+1 end n=n+1;e=l[n];else a=0;while a>-1 do if 2>=a then if a<=0 then o=e;else if a>-2 then for e=34,84 do if 2>a then y=d;break;end;r=f;break;end;else r=f;end end else if 5>a then if a==4 then h=o[y];else s=o[r];end else if a>5 then a=-2;else t(h,s);end end end a=a+1 end n=n+1;e=l[n];end end end end end else for a=0,1 do if a~=-4 then for o=28,84 do if 1~=a then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;if(t[e[d]]==t[e[h]])then n=n+1;else n=e[f];end;break;end;else if(t[e[d]]==t[e[h]])then n=n+1;else n=e[f];end;end end end end else if 107<=a then if a>107 then if 106~=a then repeat if 108~=a then local e=e[d]t[e]=t[e](t[e+1])break;end;t[e[d]]=t[e[f]]%t[e[h]];until true;else t[e[d]]=t[e[f]]%t[e[h]];end else t[e[d]][e[f]]=t[e[h]];end else if 101<=a then repeat if 106~=a then local n=e[d]t[n]=t[n](p(t,n+1,e[f]))break;end;local a;for o=0,2 do if 0>=o then a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];else if o>=-1 then repeat if 1~=o then t[e[d]][t[e[f]]]=t[e[h]];break;end;t[e[d]]=t[e[f]]-e[h];n=n+1;e=l[n];until true;else t[e[d]]=t[e[f]]-e[h];n=n+1;e=l[n];end end end until true;else local n=e[d]t[n]=t[n](p(t,n+1,e[f]))end end end end else if a<=85 then if a<=80 then if 79<=a then if a>=77 then for o=13,57 do if a<80 then for a=0,6 do if 3>a then if 1>a then t[e[d]]=s[e[f]];n=n+1;e=l[n];else if a~=-2 then for o=33,62 do if a>1 then t[e[d]]=y[e[f]];n=n+1;e=l[n];break;end;t[e[d]]=y[e[f]];n=n+1;e=l[n];break;end;else t[e[d]]=y[e[f]];n=n+1;e=l[n];end end else if a>=5 then if a~=6 then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];else t[e[d]]={};end else if a>=2 then for o=48,54 do if 3~=a then t[e[d]]=y[e[f]];n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;else t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];end end end end break;end;local e=e[d]local d,n=u(t[e](t[e+1]))r=n+e-1 local n=0;for e=e,r do n=n+1;t[e]=d[n];end;break;end;else local e=e[d]local d,n=u(t[e](t[e+1]))r=n+e-1 local n=0;for e=e,r do n=n+1;t[e]=d[n];end;end else if a~=77 then local n=e[d];local d=t[e[f]];t[n+1]=d;t[n]=d[e[h]];else if t[e[d]]then n=n+1;else n=e[f];end;end end else if 83>a then if 80~=a then repeat if 81<a then local n=e[d]t[n]=t[n](p(t,n+1,e[f]))break;end;local e=e[d];local n=t[e];for e=e+1,r do o.xKaHTdgG(n,t[e])end;until true;else local e=e[d];local n=t[e];for e=e+1,r do o.xKaHTdgG(n,t[e])end;end else if a<84 then local j,a,c,b,u,k,a,a,o,s,r,y,h;for a=0,6 do if a>=3 then if a>4 then if 2<=a then for p=31,86 do if 5<a then a=0;while a>-1 do if 3>a then if a>=1 then if 2>a then s=d;else r=f;end else o=e;end else if a>4 then if a~=1 then repeat if 6~=a then t(h,y);break;end;a=-2;until true;else t(h,y);end else if 0<=a then for e=26,59 do if a>3 then h=o[s];break;end;y=o[r];break;end;else h=o[s];end end end a=a+1 end break;end;a=0;while a>-1 do if 2<a then if a<=4 then if a>3 then h=o[s];else y=o[r];end else if a>1 then repeat if 6~=a then t(h,y);break;end;a=-2;until true;else a=-2;end end else if a<1 then o=e;else if 2~=a then s=d;else r=f;end end end a=a+1 end n=n+1;e=l[n];break;end;else a=0;while a>-1 do if 2<a then if a<=4 then if a>3 then h=o[s];else y=o[r];end else if a>1 then repeat if 6~=a then t(h,y);break;end;a=-2;until true;else a=-2;end end else if a<1 then o=e;else if 2~=a then s=d;else r=f;end end end a=a+1 end n=n+1;e=l[n];end else if 0~=a then repeat if a~=3 then a=0;while a>-1 do if 2<a then if a>=5 then if 5==a then t(h,y);else a=-2;end else if 4>a then y=o[r];else h=o[s];end end else if a<=0 then o=e;else if a>=-2 then repeat if 2>a then s=d;break;end;r=f;until true;else r=f;end end end a=a+1 end n=n+1;e=l[n];break;end;a=0;while a>-1 do if a>2 then if a<=4 then if a~=1 then repeat if a>3 then h=o[s];break;end;y=o[r];until true;else y=o[r];end else if a~=2 then for e=16,86 do if 5~=a then a=-2;break;end;t(h,y);break;end;else a=-2;end end else if a>=1 then if-2~=a then for e=38,96 do if a~=2 then s=d;break;end;r=f;break;end;else s=d;end else o=e;end end a=a+1 end n=n+1;e=l[n];until true;else a=0;while a>-1 do if a>2 then if a<=4 then if a~=1 then repeat if a>3 then h=o[s];break;end;y=o[r];until true;else y=o[r];end else if a~=2 then for e=16,86 do if 5~=a then a=-2;break;end;t(h,y);break;end;else a=-2;end end else if a>=1 then if-2~=a then for e=38,96 do if a~=2 then s=d;break;end;r=f;break;end;else s=d;end else o=e;end end a=a+1 end n=n+1;e=l[n];end end else if 1<=a then if a~=1 then a=0;while a>-1 do if a<=2 then if 1>a then o=e;else if 2>a then s=d;else r=f;end end else if 4>=a then if 4~=a then y=o[r];else h=o[s];end else if a>=4 then repeat if a<6 then t(h,y);break;end;a=-2;until true;else a=-2;end end end a=a+1 end n=n+1;e=l[n];else a=0;while a>-1 do if a<4 then if a<=1 then if a>-3 then for n=49,63 do if a>0 then c=d;break;end;o=e;break;end;else o=e;end else if-1~=a then for e=47,57 do if a~=2 then u=t;break;end;b=f;break;end;else u=t;end end else if 5<a then if a>=3 then for e=19,67 do if a~=6 then a=-2;break;end;t[h]=k;break;end;else t[h]=k;end else if 0<a then for e=15,52 do if a<5 then k=u[o[b]];break;end;h=o[c];break;end;else k=u[o[b]];end end end a=a+1 end n=n+1;e=l[n];end else j=e[d]t[j]=t[j](p(t,j+1,e[f]))n=n+1;e=l[n];end end end else if a~=81 then repeat if 85~=a then local a;for o=0,6 do if 2>=o then if o>0 then if o~=-2 then repeat if o~=2 then t(e[d],e[f]);n=n+1;e=l[n];break;end;a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];until true;else a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];end else t(e[d],e[f]);n=n+1;e=l[n];end else if o>4 then if o<6 then t(e[d],e[f]);n=n+1;e=l[n];else t(e[d],e[f]);end else if 1~=o then for a=34,58 do if 4~=o then t[e[d]]={};n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]];n=n+1;e=l[n];break;end;else t[e[d]]={};n=n+1;e=l[n];end end end end break;end;if(t[e[d]]==t[e[h]])then n=n+1;else n=e[f];end;until true;else local a;for o=0,6 do if 2>=o then if o>0 then if o~=-2 then repeat if o~=2 then t(e[d],e[f]);n=n+1;e=l[n];break;end;a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];until true;else a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];end else t(e[d],e[f]);n=n+1;e=l[n];end else if o>4 then if o<6 then t(e[d],e[f]);n=n+1;e=l[n];else t(e[d],e[f]);end else if 1~=o then for a=34,58 do if 4~=o then t[e[d]]={};n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]];n=n+1;e=l[n];break;end;else t[e[d]]={};n=n+1;e=l[n];end end end end end end end end else if 91>a then if a<88 then if a>=83 then repeat if 86~=a then local n=e[d];do return t[n](p(t,n+1,e[f]))end;break;end;local y,s,r,a,o,p,l;local n=0;while n>-1 do if 3>n then if n<1 then y=d;s=f;r=h;else if 1==n then a=e;else o=a[s];end end else if n>4 then if 4<=n then repeat if n<6 then t[p]=l;break;end;n=-2;until true;else t[p]=l;end else if 0<=n then for e=26,90 do if n>3 then l=t[o];for e=1+o,a[r]do l=l..t[e];end;break;end;p=a[y];break;end;else l=t[o];for e=1+o,a[r]do l=l..t[e];end;end end end n=n+1 end until true;else local n=e[d];do return t[n](p(t,n+1,e[f]))end;end else if a>88 then if 86<=a then repeat if 89<a then local a;t[e[d]]=t[e[f]];n=n+1;e=l[n];a=e[d]t[a](t[a+1])n=n+1;e=l[n];t[e[d]]=s[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;break;end;if(t[e[d]]==e[h])then n=n+1;else n=e[f];end;until true;else local a;t[e[d]]=t[e[f]];n=n+1;e=l[n];a=e[d]t[a](t[a+1])n=n+1;e=l[n];t[e[d]]=s[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;end else if(t[e[d]]~=e[h])then n=n+1;else n=e[f];end;end end else if 93<=a then if a>93 then if 92~=a then repeat if a<95 then local o;for a=0,6 do if 2>=a then if 0>=a then t[e[d]]=t[e[f]];n=n+1;e=l[n];else if 0<a then repeat if a<2 then t(e[d],e[f]);n=n+1;e=l[n];break;end;t(e[d],e[f]);n=n+1;e=l[n];until true;else t(e[d],e[f]);n=n+1;e=l[n];end end else if 5>a then if a~=3 then t(e[d],e[f]);n=n+1;e=l[n];else t(e[d],e[f]);n=n+1;e=l[n];end else if a~=1 then for h=41,75 do if 6~=a then t(e[d],e[f]);n=n+1;e=l[n];break;end;o=e[d]t[o]=t[o](p(t,o+1,e[f]))break;end;else o=e[d]t[o]=t[o](p(t,o+1,e[f]))end end end end break;end;do return end;until true;else local a;for o=0,6 do if 2>=o then if 0>=o then t[e[d]]=t[e[f]];n=n+1;e=l[n];else if 0<o then repeat if o<2 then t(e[d],e[f]);n=n+1;e=l[n];break;end;t(e[d],e[f]);n=n+1;e=l[n];until true;else t(e[d],e[f]);n=n+1;e=l[n];end end else if 5>o then if o~=3 then t(e[d],e[f]);n=n+1;e=l[n];else t(e[d],e[f]);n=n+1;e=l[n];end else if o~=1 then for h=41,75 do if 6~=o then t(e[d],e[f]);n=n+1;e=l[n];break;end;a=e[d]t[a]=t[a](p(t,a+1,e[f]))break;end;else a=e[d]t[a]=t[a](p(t,a+1,e[f]))end end end end end else local a,o,p;for h=0,2 do if 1<=h then if-1<=h then for r=17,90 do if 2~=h then t(e[d],e[f]);n=n+1;e=l[n];break;end;a=e[d];o=t[a]p=t[a+2];if(p>0)then if(o>t[a+1])then n=e[f];else t[a+3]=o;end elseif(o<t[a+1])then n=e[f];else t[a+3]=o;end break;end;else t(e[d],e[f]);n=n+1;e=l[n];end else t(e[d],e[f]);n=n+1;e=l[n];end end end else if a>88 then for o=46,79 do if a~=91 then t[e[d]]=t[e[f]]*e[h];break;end;local o,r,p,h,y,a;for a=0,6 do if 2<a then if a>=5 then if 6~=a then t[e[d]]={};n=n+1;e=l[n];else a=0;while a>-1 do if a>2 then if 4<a then if 6>a then t(y,h);else a=-2;end else if a>=-1 then for e=11,90 do if 4>a then h=o[p];break;end;y=o[r];break;end;else h=o[p];end end else if a<1 then o=e;else if a~=0 then repeat if 1~=a then p=f;break;end;r=d;until true;else r=d;end end end a=a+1 end end else if-1<=a then for f=39,91 do if a~=4 then t[e[d]]={};n=n+1;e=l[n];break;end;t[e[d]]={};n=n+1;e=l[n];break;end;else t[e[d]]={};n=n+1;e=l[n];end end else if a>0 then if a==1 then s[e[f]]=t[e[d]];n=n+1;e=l[n];else t[e[d]]=s[e[f]];n=n+1;e=l[n];end else t[e[d]]=(e[f]~=0);n=n+1;e=l[n];end end end break;end;else local o,r,p,h,y,a;for a=0,6 do if 2<a then if a>=5 then if 6~=a then t[e[d]]={};n=n+1;e=l[n];else a=0;while a>-1 do if a>2 then if 4<a then if 6>a then t(y,h);else a=-2;end else if a>=-1 then for e=11,90 do if 4>a then h=o[p];break;end;y=o[r];break;end;else h=o[p];end end else if a<1 then o=e;else if a~=0 then repeat if 1~=a then p=f;break;end;r=d;until true;else r=d;end end end a=a+1 end end else if-1<=a then for f=39,91 do if a~=4 then t[e[d]]={};n=n+1;e=l[n];break;end;t[e[d]]={};n=n+1;e=l[n];break;end;else t[e[d]]={};n=n+1;e=l[n];end end else if a>0 then if a==1 then s[e[f]]=t[e[d]];n=n+1;e=l[n];else t[e[d]]=s[e[f]];n=n+1;e=l[n];end else t[e[d]]=(e[f]~=0);n=n+1;e=l[n];end end end end end end end end else if 133<a then if 143>=a then if a<=138 then if a>=136 then if a<=136 then local n=e[d]t[n](p(t,n+1,e[f]))else if a>=133 then repeat if a~=137 then local o;for a=0,6 do if 2>=a then if a<1 then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];else if a>=-3 then repeat if 2~=a then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];until true;else t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];end end else if 4<a then if 6==a then if(t[e[d]]==e[h])then n=n+1;else n=e[f];end;else t[e[d]]=#t[e[f]];n=n+1;e=l[n];end else if a>=-1 then for p=32,74 do if a~=4 then o=e[d]t[o]=t[o](t[o+1])n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;else t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];end end end end break;end;t[e[d]]=t[e[f]]+e[h];until true;else local o;for a=0,6 do if 2>=a then if a<1 then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];else if a>=-3 then repeat if 2~=a then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];until true;else t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];end end else if 4<a then if 6==a then if(t[e[d]]==e[h])then n=n+1;else n=e[f];end;else t[e[d]]=#t[e[f]];n=n+1;e=l[n];end else if a>=-1 then for p=32,74 do if a~=4 then o=e[d]t[o]=t[o](t[o+1])n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;else t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];end end end end end end else if a==135 then local n=e[d];local d=t[n];for e=n+1,e[f]do o.xKaHTdgG(d,t[e])end;else t[e[d]]={};end end else if a>=141 then if a>141 then if a~=143 then local f,p,h,a,y;for s=0,1 do if 1~=s then f=e[d]p,h=u(t[f](t[f+1]))r=h+f-1 a=0;for e=f,r do a=a+1;t[e]=p[a];end;n=n+1;e=l[n];else f=e[d];y=t[f];for e=f+1,r do o.xKaHTdgG(y,t[e])end;end end else local e=e[d]t[e]=t[e](p(t,e+1,r))end else local e=e[d];do return p(t,e,r)end;end else if 135<=a then repeat if a<140 then t[e[d]]={};break;end;local a,s,y,r,a,a,k,o,u,b,c,j,h;for a=0,6 do if a>=3 then if a<=4 then if a~=1 then for p=30,72 do if a~=4 then a=0;while a>-1 do if a>2 then if 5<=a then if a>4 then repeat if 6>a then t(h,r);break;end;a=-2;until true;else t(h,r);end else if a>-1 then repeat if a>3 then h=o[s];break;end;r=o[y];until true;else h=o[s];end end else if a>=1 then if a~=-3 then repeat if a>1 then y=f;break;end;s=d;until true;else y=f;end else o=e;end end a=a+1 end n=n+1;e=l[n];break;end;a=0;while a>-1 do if a<=2 then if a<1 then o=e;else if 1~=a then y=f;else s=d;end end else if a<5 then if a==4 then h=o[s];else r=o[y];end else if 3<a then for e=38,78 do if 6~=a then t(h,r);break;end;a=-2;break;end;else a=-2;end end end a=a+1 end n=n+1;e=l[n];break;end;else a=0;while a>-1 do if a>2 then if 5<=a then if a>4 then repeat if 6>a then t(h,r);break;end;a=-2;until true;else t(h,r);end else if a>-1 then repeat if a>3 then h=o[s];break;end;r=o[y];until true;else h=o[s];end end else if a>=1 then if a~=-3 then repeat if a>1 then y=f;break;end;s=d;until true;else y=f;end else o=e;end end a=a+1 end n=n+1;e=l[n];end else if 4~=a then for r=25,55 do if 5~=a then a=0;while a>-1 do if 3>=a then if a<=1 then if a>=-2 then for n=48,68 do if a<1 then o=e;break;end;u=d;break;end;else o=e;end else if a==3 then c=t;else b=f;end end else if 5>=a then if 0<a then for e=39,82 do if 4~=a then h=o[u];break;end;j=c[o[b]];break;end;else h=o[u];end else if 6~=a then a=-2;else t[h]=j;end end end a=a+1 end break;end;k=e[d]t[k]=t[k](p(t,k+1,e[f]))n=n+1;e=l[n];break;end;else k=e[d]t[k]=t[k](p(t,k+1,e[f]))n=n+1;e=l[n];end end else if a<=0 then a=0;while a>-1 do if a>=3 then if a>4 then if 1<a then repeat if a~=6 then t(h,r);break;end;a=-2;until true;else a=-2;end else if a==3 then r=o[y];else h=o[s];end end else if 1>a then o=e;else if 1~=a then y=f;else s=d;end end end a=a+1 end n=n+1;e=l[n];else if a~=0 then repeat if a>1 then a=0;while a>-1 do if 2>=a then if 0<a then if a==2 then y=f;else s=d;end else o=e;end else if a<5 then if 1<a then repeat if 3~=a then h=o[s];break;end;r=o[y];until true;else r=o[y];end else if 4~=a then for e=21,58 do if 6>a then t(h,r);break;end;a=-2;break;end;else t(h,r);end end end a=a+1 end n=n+1;e=l[n];break;end;a=0;while a>-1 do if 2<a then if 4<a then if 1<=a then repeat if 5<a then a=-2;break;end;t(h,r);until true;else t(h,r);end else if a>=1 then for e=12,70 do if 3~=a then h=o[s];break;end;r=o[y];break;end;else h=o[s];end end else if a<=0 then o=e;else if a==1 then s=d;else y=f;end end end a=a+1 end n=n+1;e=l[n];until true;else a=0;while a>-1 do if 2>=a then if 0<a then if a==2 then y=f;else s=d;end else o=e;end else if a<5 then if 1<a then repeat if 3~=a then h=o[s];break;end;r=o[y];until true;else r=o[y];end else if 4~=a then for e=21,58 do if 6>a then t(h,r);break;end;a=-2;break;end;else t(h,r);end end end a=a+1 end n=n+1;e=l[n];end end end end until true;else local a,s,y,r,a,a,k,o,u,j,b,c,h;for a=0,6 do if a>=3 then if a<=4 then if a~=1 then for p=30,72 do if a~=4 then a=0;while a>-1 do if a>2 then if 5<=a then if a>4 then repeat if 6>a then t(h,r);break;end;a=-2;until true;else t(h,r);end else if a>-1 then repeat if a>3 then h=o[s];break;end;r=o[y];until true;else h=o[s];end end else if a>=1 then if a~=-3 then repeat if a>1 then y=f;break;end;s=d;until true;else y=f;end else o=e;end end a=a+1 end n=n+1;e=l[n];break;end;a=0;while a>-1 do if a<=2 then if a<1 then o=e;else if 1~=a then y=f;else s=d;end end else if a<5 then if a==4 then h=o[s];else r=o[y];end else if 3<a then for e=38,78 do if 6~=a then t(h,r);break;end;a=-2;break;end;else a=-2;end end end a=a+1 end n=n+1;e=l[n];break;end;else a=0;while a>-1 do if a>2 then if 5<=a then if a>4 then repeat if 6>a then t(h,r);break;end;a=-2;until true;else t(h,r);end else if a>-1 then repeat if a>3 then h=o[s];break;end;r=o[y];until true;else h=o[s];end end else if a>=1 then if a~=-3 then repeat if a>1 then y=f;break;end;s=d;until true;else y=f;end else o=e;end end a=a+1 end n=n+1;e=l[n];end else if 4~=a then for r=25,55 do if 5~=a then a=0;while a>-1 do if 3>=a then if a<=1 then if a>=-2 then for n=48,68 do if a<1 then o=e;break;end;u=d;break;end;else o=e;end else if a==3 then b=t;else j=f;end end else if 5>=a then if 0<a then for e=39,82 do if 4~=a then h=o[u];break;end;c=b[o[j]];break;end;else h=o[u];end else if 6~=a then a=-2;else t[h]=c;end end end a=a+1 end break;end;k=e[d]t[k]=t[k](p(t,k+1,e[f]))n=n+1;e=l[n];break;end;else k=e[d]t[k]=t[k](p(t,k+1,e[f]))n=n+1;e=l[n];end end else if a<=0 then a=0;while a>-1 do if a>=3 then if a>4 then if 1<a then repeat if a~=6 then t(h,r);break;end;a=-2;until true;else a=-2;end else if a==3 then r=o[y];else h=o[s];end end else if 1>a then o=e;else if 1~=a then y=f;else s=d;end end end a=a+1 end n=n+1;e=l[n];else if a~=0 then repeat if a>1 then a=0;while a>-1 do if 2>=a then if 0<a then if a==2 then y=f;else s=d;end else o=e;end else if a<5 then if 1<a then repeat if 3~=a then h=o[s];break;end;r=o[y];until true;else r=o[y];end else if 4~=a then for e=21,58 do if 6>a then t(h,r);break;end;a=-2;break;end;else t(h,r);end end end a=a+1 end n=n+1;e=l[n];break;end;a=0;while a>-1 do if 2<a then if 4<a then if 1<=a then repeat if 5<a then a=-2;break;end;t(h,r);until true;else t(h,r);end else if a>=1 then for e=12,70 do if 3~=a then h=o[s];break;end;r=o[y];break;end;else h=o[s];end end else if a<=0 then o=e;else if a==1 then s=d;else y=f;end end end a=a+1 end n=n+1;e=l[n];until true;else a=0;while a>-1 do if 2>=a then if 0<a then if a==2 then y=f;else s=d;end else o=e;end else if a<5 then if 1<a then repeat if 3~=a then h=o[s];break;end;r=o[y];until true;else r=o[y];end else if 4~=a then for e=21,58 do if 6>a then t(h,r);break;end;a=-2;break;end;else t(h,r);end end end a=a+1 end n=n+1;e=l[n];end end end end end end end else if a<=148 then if a>145 then if 146>=a then for e=e[d],e[f]do t[e]=nil;end;else if a>=144 then repeat if a>147 then local a;t[e[d]][e[f]]=t[e[h]];n=n+1;e=l[n];a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];t[e[d]]=s[e[f]];n=n+1;e=l[n];t[e[d]]=y[e[f]];n=n+1;e=l[n];t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];a=e[d]t[a](p(t,a+1,e[f]))break;end;local n=e[d]local d,e=u(t[n](p(t,n+1,e[f])))r=e+n-1 local e=0;for n=n,r do e=e+1;t[n]=d[e];end;until true;else local n=e[d]local d,e=u(t[n](p(t,n+1,e[f])))r=e+n-1 local e=0;for n=n,r do e=e+1;t[n]=d[e];end;end end else if a>=140 then repeat if a>144 then t[e[d]]();break;end;local e=e[d]t[e]=t[e](p(t,e+1,r))until true;else local e=e[d]t[e]=t[e](p(t,e+1,r))end end else if a<=150 then if 149~=a then for e=e[d],e[f]do t[e]=nil;end;else local l,a,o,p,r,h;local n=0;while n>-1 do if 3<n then if n<=5 then if n>2 then repeat if n<5 then r=p[l[o]];break;end;h=l[a];until true;else h=l[a];end else if n>5 then for e=33,91 do if n~=7 then t[h]=r;break;end;n=-2;break;end;else n=-2;end end else if 2>n then if n>=-1 then repeat if 0~=n then a=d;break;end;l=e;until true;else a=d;end else if n>-1 then repeat if 3~=n then o=f;break;end;p=t;until true;else o=f;end end end n=n+1 end end else if a>=152 then if 148<a then repeat if a>152 then local e=e[d];local n=t[e];for e=e+1,r do o.xKaHTdgG(n,t[e])end;break;end;t[e[d]]=t[e[f]][e[h]];n=n+1;e=l[n];t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];t[e[d]]=t[e[f]][e[h]];n=n+1;e=l[n];t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];t[e[d]]=(e[f]~=0);n=n+1;e=l[n];t[e[d]]=y[e[f]];n=n+1;e=l[n];t[e[d]]=t[e[f]][e[h]];until true;else local e=e[d];local n=t[e];for e=e+1,r do o.xKaHTdgG(n,t[e])end;end else local r,s,p,y,u,a,o,h,k;for a=0,2 do if a<=0 then t[e[d]]=#t[e[f]];n=n+1;e=l[n];else if a<2 then a=0;while a>-1 do if 2<a then if 4>=a then if 0<a then repeat if 4>a then y=r[p];break;end;u=r[s];until true;else y=r[p];end else if 5==a then t(u,y);else a=-2;end end else if a<1 then r=e;else if a>-3 then repeat if a~=2 then s=d;break;end;p=f;until true;else p=f;end end end a=a+1 end n=n+1;e=l[n];else o=e[d];h=t[o]k=t[o+2];if(k>0)then if(h>t[o+1])then n=e[f];else t[o+3]=h;end elseif(h<t[o+1])then n=e[f];else t[o+3]=h;end end end end end end end end else if a>123 then if a<=128 then if 126>a then if a~=122 then repeat if 124<a then s[e[f]]=t[e[d]];break;end;t[e[d]]=t[e[f]]*e[h];until true;else s[e[f]]=t[e[d]];end else if 127<=a then if 127~=a then local a;for o=0,1 do if o~=-1 then for h=27,61 do if 0~=o then if not t[e[d]]then n=n+1;else n=e[f];end;break;end;a=e[d]t[a]=t[a]()n=n+1;e=l[n];break;end;else if not t[e[d]]then n=n+1;else n=e[f];end;end end else local o;for a=0,6 do if 2>=a then if 1>a then t(e[d],e[f]);n=n+1;e=l[n];else if a~=2 then o=e[d]t[o]=t[o](p(t,o+1,e[f]))n=n+1;e=l[n];else t[e[d]]=t[e[f]];n=n+1;e=l[n];end end else if a<=4 then if 3==a then t(e[d],e[f]);n=n+1;e=l[n];else t(e[d],e[f]);n=n+1;e=l[n];end else if a==5 then t(e[d],e[f]);n=n+1;e=l[n];else t(e[d],e[f]);end end end end end else for a=0,1 do if a>-3 then for o=34,65 do if a>0 then if not t[e[d]]then n=n+1;else n=e[f];end;break;end;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;else if not t[e[d]]then n=n+1;else n=e[f];end;end end end end else if a>130 then if a>131 then if 132~=a then if t[e[d]]then n=n+1;else n=e[f];end;else if not t[e[d]]then n=n+1;else n=e[f];end;end else local p,y,u,s,k,r,a,o;t[e[d]]=t[e[f]][e[h]];n=n+1;e=l[n];a=0;while a>-1 do if a<4 then if a<=1 then if a>=-4 then repeat if a<1 then p=e;break;end;y=d;until true;else y=d;end else if a~=0 then for e=47,94 do if a>2 then s=t;break;end;u=f;break;end;else s=t;end end else if a<6 then if a==5 then r=p[y];else k=s[p[u]];end else if 3~=a then repeat if 6<a then a=-2;break;end;t[r]=k;until true;else t[r]=k;end end end a=a+1 end n=n+1;e=l[n];o=e[d]t[o]=t[o](t[o+1])n=n+1;e=l[n];t[e[d]][t[e[f]]]=t[e[h]];n=n+1;e=l[n];t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];t[e[d]][t[e[f]]]=t[e[h]];end else if a>127 then repeat if a~=129 then local p,r,o,y,s,k,a;t[e[d]]=t[e[f]][e[h]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];a=0;while a>-1 do if 4<=a then if a<=5 then if a==5 then k=p[r];else s=y[p[o]];end else if 4~=a then repeat if a~=7 then t[k]=s;break;end;a=-2;until true;else a=-2;end end else if a>=2 then if a>=-2 then for e=34,82 do if a~=3 then o=f;break;end;y=t;break;end;else o=f;end else if 0==a then p=e;else r=d;end end end a=a+1 end n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;break;end;t[e[d]]=t[e[f]];n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);until true;else t[e[d]]=t[e[f]];n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);end end end else if a>=119 then if a>120 then if a>121 then if 121~=a then repeat if a>122 then local a,o,h;for p=0,2 do if 0>=p then t[e[d]]=#t[e[f]];n=n+1;e=l[n];else if-3~=p then for r=15,94 do if p~=2 then t(e[d],e[f]);n=n+1;e=l[n];break;end;a=e[d];o=t[a]h=t[a+2];if(h>0)then if(o>t[a+1])then n=e[f];else t[a+3]=o;end elseif(o<t[a+1])then n=e[f];else t[a+3]=o;end break;end;else a=e[d];o=t[a]h=t[a+2];if(h>0)then if(o>t[a+1])then n=e[f];else t[a+3]=o;end elseif(o<t[a+1])then n=e[f];else t[a+3]=o;end end end end break;end;t[e[d]][e[f]]=t[e[h]];until true;else local a,o,h;for p=0,2 do if 0>=p then t[e[d]]=#t[e[f]];n=n+1;e=l[n];else if-3~=p then for r=15,94 do if p~=2 then t(e[d],e[f]);n=n+1;e=l[n];break;end;a=e[d];o=t[a]h=t[a+2];if(h>0)then if(o>t[a+1])then n=e[f];else t[a+3]=o;end elseif(o<t[a+1])then n=e[f];else t[a+3]=o;end break;end;else a=e[d];o=t[a]h=t[a+2];if(h>0)then if(o>t[a+1])then n=e[f];else t[a+3]=o;end elseif(o<t[a+1])then n=e[f];else t[a+3]=o;end end end end end else t[e[d]]=y[e[f]];end else if a>119 then if(t[e[d]]~=e[h])then n=n+1;else n=e[f];end;else if(t[e[d]]==t[e[h]])then n=n+1;else n=e[f];end;end end else if a>116 then if a~=116 then for o=42,62 do if 118~=a then local l,a,h,p,o;local n=0;while n>-1 do if n>2 then if 5>n then if-1<n then for e=14,86 do if 3~=n then o=l[a];break;end;p=l[h];break;end;else o=l[a];end else if 1~=n then repeat if 6>n then t(o,p);break;end;n=-2;until true;else n=-2;end end else if n>0 then if-3<n then for e=44,83 do if n~=1 then h=f;break;end;a=d;break;end;else a=d;end else l=e;end end n=n+1 end break;end;local a,o;t[e[d]]=#t[e[f]];n=n+1;e=l[n];t[e[d]]=t[e[f]]%t[e[h]];n=n+1;e=l[n];t[e[d]]=t[e[f]]+e[h];n=n+1;e=l[n];t[e[d]]=y[e[f]];n=n+1;e=l[n];a=e[d];o=t[e[f]];t[a+1]=o;t[a]=o[e[h]];n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t[e[d]]=t[e[f]];break;end;else local l,a,h,p,o;local n=0;while n>-1 do if n>2 then if 5>n then if-1<n then for e=14,86 do if 3~=n then o=l[a];break;end;p=l[h];break;end;else o=l[a];end else if 1~=n then repeat if 6>n then t(o,p);break;end;n=-2;until true;else n=-2;end end else if n>0 then if-3<n then for e=44,83 do if n~=1 then h=f;break;end;a=d;break;end;else a=d;end else l=e;end end n=n+1 end end else if a>=111 then repeat if a~=116 then local a;t[e[d]]=t[e[f]];n=n+1;e=l[n];a=e[d]t[a](t[a+1])n=n+1;e=l[n];t[e[d]]=s[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;n=n+1;e=l[n];for e=e[d],e[f]do t[e]=nil;end;break;end;t[e[d]]=c(j[e[f]],nil,s);until true;else t[e[d]]=c(j[e[f]],nil,s);end end end end end end else if 38<=a then if 56<a then if 67<=a then if a<=71 then if 69>a then if 68>a then local e=e[d]t[e](t[e+1])else t[e[d]]=(e[f]~=0);end else if a<=69 then local o,y,s,r,p,a;for a=0,3 do if 1>=a then if 0==a then a=0;while a>-1 do if 2<a then if 4<a then if 1<a then for e=23,84 do if 5<a then a=-2;break;end;t(p,r);break;end;else t(p,r);end else if 4~=a then r=o[s];else p=o[y];end end else if 0>=a then o=e;else if a>0 then repeat if 1~=a then s=f;break;end;y=d;until true;else y=d;end end end a=a+1 end n=n+1;e=l[n];else a=0;while a>-1 do if a<3 then if a<1 then o=e;else if a~=-2 then for e=35,76 do if a<2 then y=d;break;end;s=f;break;end;else s=f;end end else if a>4 then if a>2 then repeat if a<6 then t(p,r);break;end;a=-2;until true;else a=-2;end else if a~=3 then p=o[y];else r=o[s];end end end a=a+1 end n=n+1;e=l[n];end else if a>=1 then repeat if 3>a then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;if not t[e[d]]then n=n+1;else n=e[f];end;until true;else if not t[e[d]]then n=n+1;else n=e[f];end;end end end else if 68~=a then for o=24,98 do if 71~=a then local h,o,k,y,r,a,p;a=0;while a>-1 do if 3>a then if a>=1 then if-1<=a then for e=18,85 do if 2>a then o=d;break;end;k=f;break;end;else o=d;end else h=e;end else if 4>=a then if a>0 then for e=30,54 do if a<4 then y=h[k];break;end;r=h[o];break;end;else r=h[o];end else if a==6 then a=-2;else t(r,y);end end end a=a+1 end n=n+1;e=l[n];p=e[d]t[p](t[p+1])n=n+1;e=l[n];t[e[d]]=s[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;n=n+1;e=l[n];for e=e[d],e[f]do t[e]=nil;end;break;end;t[e[d]]=t[e[f]]%t[e[h]];break;end;else t[e[d]]=t[e[f]]%t[e[h]];end end end else if a>73 then if a>74 then if a~=75 then local s,r,p,o,y,a;t[e[d]]=t[e[f]][e[h]];n=n+1;e=l[n];t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];t[e[d]]=t[e[f]][e[h]];n=n+1;e=l[n];t[e[d]][t[e[f]]]=t[e[h]];n=n+1;e=l[n];do return t[e[d]]end n=n+1;e=l[n];s=e[d];r={};for e=1,#k do p=k[e];for e=0,#p do o=p[e];y=o[1];a=o[2];if y==t and a>=s then r[a]=y[a];o[1]=r;end;end;end;else local l=e[d];local d={};for e=1,#k do local e=k[e];for n=0,#e do local e=e[n];local f=e[1];local n=e[2];if f==t and n>=l then d[n]=f[n];e[1]=d;end;end;end;end else t[e[d]]=c(j[e[f]],nil,s);end else if 72<a then t[e[d]]=(e[f]~=0);else local n=e[d];local d=t[n];for e=n+1,e[f]do o.xKaHTdgG(d,t[e])end;end end end else if 62>a then if a<59 then if 55<=a then repeat if a~=57 then local a,k,y,o;for h=0,5 do if h<3 then if 1>h then a=e[d]t[a]=t[a](t[a+1])n=n+1;e=l[n];else if h==2 then t(e[d],e[f]);n=n+1;e=l[n];else a=e[d]t[a]=t[a]()n=n+1;e=l[n];end end else if h<=3 then t[e[d]]=s[e[f]];n=n+1;e=l[n];else if 3<=h then for s=15,58 do if 5>h then a=e[d]k,y=u(t[a](p(t,a+1,e[f])))r=y+a-1 o=0;for e=a,r do o=o+1;t[e]=k[o];end;n=n+1;e=l[n];break;end;a=e[d]t[a]=t[a](p(t,a+1,r))break;end;else a=e[d]k,y=u(t[a](p(t,a+1,e[f])))r=y+a-1 o=0;for e=a,r do o=o+1;t[e]=k[o];end;n=n+1;e=l[n];end end end end break;end;if not t[e[d]]then n=n+1;else n=e[f];end;until true;else local a,k,y,o;for h=0,5 do if h<3 then if 1>h then a=e[d]t[a]=t[a](t[a+1])n=n+1;e=l[n];else if h==2 then t(e[d],e[f]);n=n+1;e=l[n];else a=e[d]t[a]=t[a]()n=n+1;e=l[n];end end else if h<=3 then t[e[d]]=s[e[f]];n=n+1;e=l[n];else if 3<=h then for s=15,58 do if 5>h then a=e[d]k,y=u(t[a](p(t,a+1,e[f])))r=y+a-1 o=0;for e=a,r do o=o+1;t[e]=k[o];end;n=n+1;e=l[n];break;end;a=e[d]t[a]=t[a](p(t,a+1,r))break;end;else a=e[d]k,y=u(t[a](p(t,a+1,e[f])))r=y+a-1 o=0;for e=a,r do o=o+1;t[e]=k[o];end;n=n+1;e=l[n];end end end end end else if 60<=a then if 58<=a then repeat if a~=60 then local l,p,a,o,h;local n=0;while n>-1 do if n>2 then if 5>n then if 2<n then for e=27,65 do if n~=3 then h=l[p];break;end;o=l[a];break;end;else o=l[a];end else if n~=1 then for e=44,88 do if 5~=n then n=-2;break;end;t(h,o);break;end;else n=-2;end end else if n<=0 then l=e;else if n==2 then a=f;else p=d;end end end n=n+1 end break;end;local a;t[e[d]]=t[e[f]];n=n+1;e=l[n];a=e[d]t[a](t[a+1])n=n+1;e=l[n];t[e[d]]=s[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;n=n+1;e=l[n];for e=e[d],e[f]do t[e]=nil;end;until true;else local a;t[e[d]]=t[e[f]];n=n+1;e=l[n];a=e[d]t[a](t[a+1])n=n+1;e=l[n];t[e[d]]=s[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;n=n+1;e=l[n];for e=e[d],e[f]do t[e]=nil;end;end else t[e[d]]=t[e[f]]+t[e[h]];end end else if a<=63 then if 63==a then local a;for o=0,3 do if o<2 then if-4<=o then repeat if o~=1 then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];until true;else t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];end else if o~=-1 then repeat if o<3 then a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];break;end;if not t[e[d]]then n=n+1;else n=e[f];end;until true;else a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];end end end else if(t[e[d]]==e[h])then n=n+1;else n=e[f];end;end else if 64>=a then local n=e[d]local d,e=u(t[n](p(t,n+1,e[f])))r=e+n-1 local e=0;for n=n,r do e=e+1;t[n]=d[e];end;else if 64<a then for p=35,56 do if 66~=a then local r=j[e[f]];local p;local a={};p=o.zmZVolie({},{__index=function(n,e)local e=a[e];return e[1][e[2]];end,__newindex=function(t,e,n)local e=a[e]e[1][e[2]]=n;end;});for d=1,e[h]do n=n+1;local e=l[n];if e[_]==149 then a[d-1]={t,e[f]};else a[d-1]={y,e[f]};end;k[#k+1]=a;end;t[e[d]]=c(r,p,s);break;end;local e=e[d]t[e]=t[e](t[e+1])break;end;else local e=e[d]t[e]=t[e](t[e+1])end end end end end else if 47>a then if a<42 then if 40<=a then if 39<a then for o=48,80 do if 40~=a then local a;t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);break;end;local e=e[d];do return p(t,e,r)end;break;end;else local a;t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);end else if 38<a then local d=e[d];local a=t[d+2];local l=t[d]+a;t[d]=l;if(a>0)then if(l<=t[d+1])then n=e[f];t[d+3]=l;end elseif(l>=t[d+1])then n=e[f];t[d+3]=l;end else local n=e[d];local d=t[e[f]];t[n+1]=d;t[n]=d[e[h]];end end else if a<=43 then if a~=43 then do return end;else for a=0,1 do if 0~=a then if t[e[d]]then n=n+1;else n=e[f];end;else t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];end end end else if a<=44 then local a;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];a=e[d];do return t[a](p(t,a+1,e[f]))end;n=n+1;e=l[n];a=e[d];do return p(t,a,r)end;n=n+1;e=l[n];do return end;else if 46~=a then t[e[d]]=t[e[f]]-t[e[h]];else local a,o,h;for p=0,2 do if p<=0 then t[e[d]]=#t[e[f]];n=n+1;e=l[n];else if p<2 then t(e[d],e[f]);n=n+1;e=l[n];else a=e[d];o=t[a]h=t[a+2];if(h>0)then if(o>t[a+1])then n=e[f];else t[a+3]=o;end elseif(o<t[a+1])then n=e[f];else t[a+3]=o;end end end end end end end end else if a>=52 then if 54<=a then if a<55 then local n=e[d]t[n](p(t,n+1,e[f]))else if 51~=a then for o=29,73 do if a~=56 then do return t[e[d]]end break;end;local a;t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t(e[d],e[f]);break;end;else local a;t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t(e[d],e[f]);end end else if 48~=a then repeat if 53>a then local e=e[d]t[e](t[e+1])break;end;local l,o,a,p,r,h;local n=0;while n>-1 do if n>=4 then if n<=5 then if n>=3 then for e=34,52 do if n~=5 then r=p[l[a]];break;end;h=l[o];break;end;else h=l[o];end else if n~=5 then repeat if n~=6 then n=-2;break;end;t[h]=r;until true;else n=-2;end end else if 2>n then if-4<=n then repeat if 1~=n then l=e;break;end;o=d;until true;else l=e;end else if 1~=n then for e=29,85 do if 3~=n then a=f;break;end;p=t;break;end;else a=f;end end end n=n+1 end until true;else local l,h,a,p,r,o;local n=0;while n>-1 do if n>=4 then if n<=5 then if n>=3 then for e=34,52 do if n~=5 then r=p[l[a]];break;end;o=l[h];break;end;else o=l[h];end else if n~=5 then repeat if n~=6 then n=-2;break;end;t[o]=r;until true;else n=-2;end end else if 2>n then if-4<=n then repeat if 1~=n then l=e;break;end;h=d;until true;else l=e;end else if 1~=n then for e=29,85 do if 3~=n then a=f;break;end;p=t;break;end;else a=f;end end end n=n+1 end end end else if a<=48 then if 44<a then for o=31,56 do if a~=48 then local r,k,s,p,y,a,o,h,u;for a=0,2 do if 1<=a then if a~=2 then a=0;while a>-1 do if 3>a then if a<1 then r=e;else if-2<=a then repeat if 1<a then s=f;break;end;k=d;until true;else k=d;end end else if a>=5 then if a>2 then repeat if a>5 then a=-2;break;end;t(y,p);until true;else t(y,p);end else if a>1 then repeat if 3<a then y=r[k];break;end;p=r[s];until true;else p=r[s];end end end a=a+1 end n=n+1;e=l[n];else o=e[d];h=t[o]u=t[o+2];if(u>0)then if(h>t[o+1])then n=e[f];else t[o+3]=h;end elseif(h<t[o+1])then n=e[f];else t[o+3]=h;end end else t[e[d]]=#t[e[f]];n=n+1;e=l[n];end end break;end;local a;a=e[d];do return t[a](p(t,a+1,e[f]))end;n=n+1;e=l[n];a=e[d];do return p(t,a,r)end;n=n+1;e=l[n];do return end;break;end;else local a;a=e[d];do return t[a](p(t,a+1,e[f]))end;n=n+1;e=l[n];a=e[d];do return p(t,a,r)end;n=n+1;e=l[n];do return end;end else if a>49 then if 49<=a then for o=25,66 do if 50<a then t[e[d]]=#t[e[f]];break;end;local a;t[e[d]]=t[e[f]];n=n+1;e=l[n];a=e[d]t[a](t[a+1])n=n+1;e=l[n];t[e[d]]=s[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;n=n+1;e=l[n];for e=e[d],e[f]do t[e]=nil;end;break;end;else t[e[d]]=#t[e[f]];end else local p,o,k,s,r,a,h,y,u;for a=0,2 do if a>0 then if 2~=a then a=0;while a>-1 do if a>=3 then if a>4 then if a>=4 then for e=24,72 do if 6~=a then t(r,s);break;end;a=-2;break;end;else t(r,s);end else if 0~=a then for e=38,77 do if 3<a then r=p[o];break;end;s=p[k];break;end;else r=p[o];end end else if a>=1 then if-3<a then repeat if a~=1 then k=f;break;end;o=d;until true;else o=d;end else p=e;end end a=a+1 end n=n+1;e=l[n];else h=e[d];y=t[h]u=t[h+2];if(u>0)then if(y>t[h+1])then n=e[f];else t[h+3]=y;end elseif(y<t[h+1])then n=e[f];else t[h+3]=y;end end else a=0;while a>-1 do if a>=3 then if 4>=a then if a~=4 then s=p[k];else r=p[o];end else if a<6 then t(r,s);else a=-2;end end else if a<=0 then p=e;else if a>=0 then for e=11,85 do if a~=2 then o=d;break;end;k=f;break;end;else o=d;end end end a=a+1 end n=n+1;e=l[n];end end end end end end end else if 18<a then if 27<a then if 32>=a then if a<=29 then if 28<a then local r=j[e[f]];local p;local a={};p=o.zmZVolie({},{__index=function(n,e)local e=a[e];return e[1][e[2]];end,__newindex=function(t,e,n)local e=a[e]e[1][e[2]]=n;end;});for d=1,e[h]do n=n+1;local e=l[n];if e[_]==149 then a[d-1]={t,e[f]};else a[d-1]={y,e[f]};end;k[#k+1]=a;end;t[e[d]]=c(r,p,s);else t[e[d]]=t[e[f]][e[h]];end else if 30<a then if 31==a then t[e[d]]=t[e[f]][e[h]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;else t[e[d]]=s[e[f]];end else t[e[d]]=t[e[f]][t[e[h]]];end end else if 35<=a then if a<36 then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t(e[d],e[f]);n=n+1;e=l[n];t[e[d]]=#t[e[f]];n=n+1;e=l[n];t[e[d]]=t[e[f]]-t[e[h]];n=n+1;e=l[n];t(e[d],e[f]);else if 35<a then for n=24,78 do if 36<a then local e=e[d]local d,n=u(t[e](t[e+1]))r=n+e-1 local n=0;for e=e,r do n=n+1;t[e]=d[n];end;break;end;t[e[d]]=t[e[f]]%e[h];break;end;else t[e[d]]=t[e[f]]%e[h];end end else if 29<=a then for n=39,92 do if 33<a then t[e[d]]=t[e[f]]+e[h];break;end;do return t[e[d]]end break;end;else do return t[e[d]]end end end end else if a<23 then if a<=20 then if 17<=a then repeat if 20~=a then local a,h;for r=0,1 do if-3<=r then for y=13,61 do if 1~=r then a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];break;end;a=e[d];h=t[a];for e=a+1,e[f]do o.xKaHTdgG(h,t[e])end;break;end;else a=e[d];h=t[a];for e=a+1,e[f]do o.xKaHTdgG(h,t[e])end;end end break;end;local r,b,o,j,k,s,y,u,a;r=e[d];b=t[e[f]];t[r+1]=b;t[r]=b[e[h]];n=n+1;e=l[n];a=0;while a>-1 do if a>=4 then if 5>=a then if 5>a then y=s[o[k]];else u=o[j];end else if 4<=a then repeat if 7>a then t[u]=y;break;end;a=-2;until true;else a=-2;end end else if 2<=a then if a>2 then s=t;else k=f;end else if a>=-3 then repeat if 1>a then o=e;break;end;j=d;until true;else o=e;end end end a=a+1 end n=n+1;e=l[n];a=0;while a>-1 do if 4<=a then if 6<=a then if a>=3 then repeat if 7~=a then t[u]=y;break;end;a=-2;until true;else t[u]=y;end else if a~=0 then repeat if 4<a then u=o[j];break;end;y=s[o[k]];until true;else y=s[o[k]];end end else if 1>=a then if a>=-4 then for n=45,93 do if 1>a then o=e;break;end;j=d;break;end;else o=e;end else if-2<=a then repeat if 3>a then k=f;break;end;s=t;until true;else s=t;end end end a=a+1 end n=n+1;e=l[n];r=e[d]t[r]=t[r](p(t,r+1,e[f]))n=n+1;e=l[n];t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];t[e[d]]=t[e[f]]*e[h];until true;else local a,h;for r=0,1 do if-3<=r then for y=13,61 do if 1~=r then a=e[d]t[a]=t[a](p(t,a+1,e[f]))n=n+1;e=l[n];break;end;a=e[d];h=t[a];for e=a+1,e[f]do o.xKaHTdgG(h,t[e])end;break;end;else a=e[d];h=t[a];for e=a+1,e[f]do o.xKaHTdgG(h,t[e])end;end end end else if a>19 then for o=31,90 do if a<22 then local o,r,c,j,b,u,k,a;for a=0,6 do if 2<a then if a>=5 then if 3~=a then for h=11,59 do if a>5 then o=e[d]t[o](p(t,o+1,e[f]))break;end;a=0;while a>-1 do if a>3 then if 5>=a then if a==5 then k=r[c];else u=b[r[j]];end else if 3~=a then for e=40,59 do if a<7 then t[k]=u;break;end;a=-2;break;end;else t[k]=u;end end else if a>1 then if-1<=a then repeat if a<3 then j=f;break;end;b=t;until true;else j=f;end else if a~=-3 then repeat if a~=0 then c=d;break;end;r=e;until true;else r=e;end end end a=a+1 end n=n+1;e=l[n];break;end;else o=e[d]t[o](p(t,o+1,e[f]))end else if 4==a then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];else t[e[d]]=y[e[f]];n=n+1;e=l[n];end end else if 1>a then t[e[d]][e[f]]=t[e[h]];n=n+1;e=l[n];else if a~=0 then repeat if a>1 then t[e[d]]=s[e[f]];n=n+1;e=l[n];break;end;o=e[d]t[o]=t[o](p(t,o+1,e[f]))n=n+1;e=l[n];until true;else o=e[d]t[o]=t[o](p(t,o+1,e[f]))n=n+1;e=l[n];end end end end break;end;local d=e[d];local l=t[d]local a=t[d+2];if(a>0)then if(l>t[d+1])then n=e[f];else t[d+3]=l;end elseif(l<t[d+1])then n=e[f];else t[d+3]=l;end break;end;else local d=e[d];local l=t[d]local a=t[d+2];if(a>0)then if(l>t[d+1])then n=e[f];else t[d+3]=l;end elseif(l<t[d+1])then n=e[f];else t[d+3]=l;end end end else if 25<=a then if a<26 then n=e[f];else if 27~=a then local e=e[d]t[e]=t[e]()else t[e[d]]=t[e[f]][t[e[h]]];end end else if 23==a then local d=e[d];local l=t[d]local a=t[d+2];if(a>0)then if(l>t[d+1])then n=e[f];else t[d+3]=l;end elseif(l<t[d+1])then n=e[f];else t[d+3]=l;end else local o,s,r,k,u,a,p,y,j;for a=0,4 do if 1>=a then if a~=1 then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];else a=0;while a>-1 do if a>2 then if 5>a then if a>=-1 then repeat if 3<a then u=o[s];break;end;k=o[r];until true;else k=o[r];end else if a>5 then a=-2;else t(u,k);end end else if a>=1 then if a>=0 then repeat if a~=2 then s=d;break;end;r=f;until true;else s=d;end else o=e;end end a=a+1 end n=n+1;e=l[n];end else if a>=3 then if a~=3 then p=e[d];y=t[p]j=t[p+2];if(j>0)then if(y>t[p+1])then n=e[f];else t[p+3]=y;end elseif(y<t[p+1])then n=e[f];else t[p+3]=y;end else a=0;while a>-1 do if a>2 then if a<=4 then if 3==a then k=o[r];else u=o[s];end else if a<6 then t(u,k);else a=-2;end end else if 1<=a then if a>=-3 then repeat if 2~=a then s=d;break;end;r=f;until true;else r=f;end else o=e;end end a=a+1 end n=n+1;e=l[n];end else t[e[d]]=#t[e[f]];n=n+1;e=l[n];end end end end end end end else if 9>a then if 4>a then if 1<a then if 0~=a then repeat if a<3 then t[e[d]]=t[e[f]][e[h]];break;end;t[e[d]]=s[e[f]];until true;else t[e[d]]=t[e[f]][e[h]];end else if a~=0 then local u,b,o,y,s,r,k,j,a;for a=0,5 do if a>2 then if a>3 then if 0~=a then repeat if a<5 then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]]+t[e[h]];until true;else t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];end else u=e[d]t[u]=t[u](p(t,u+1,e[f]))n=n+1;e=l[n];end else if a<1 then u=e[d];b=t[e[f]];t[u+1]=b;t[u]=b[e[h]];n=n+1;e=l[n];else if a>0 then for h=22,52 do if 2~=a then a=0;while a>-1 do if a<=3 then if a>=2 then if 0<a then repeat if a~=3 then s=f;break;end;r=t;until true;else r=t;end else if-4~=a then for n=47,94 do if a<1 then o=e;break;end;y=d;break;end;else y=d;end end else if a<=5 then if 2<a then for e=34,89 do if 4~=a then j=o[y];break;end;k=r[o[s]];break;end;else k=r[o[s]];end else if 5<=a then for e=12,83 do if a<7 then t[j]=k;break;end;a=-2;break;end;else a=-2;end end end a=a+1 end n=n+1;e=l[n];break;end;a=0;while a>-1 do if a<4 then if a<=1 then if-2~=a then repeat if 1>a then o=e;break;end;y=d;until true;else o=e;end else if 2~=a then r=t;else s=f;end end else if a<6 then if 2~=a then for e=26,85 do if 5~=a then k=r[o[s]];break;end;j=o[y];break;end;else j=o[y];end else if 2<a then for e=24,72 do if 6~=a then a=-2;break;end;t[j]=k;break;end;else a=-2;end end end a=a+1 end n=n+1;e=l[n];break;end;else a=0;while a>-1 do if a<=3 then if a>=2 then if 0<a then repeat if a~=3 then s=f;break;end;r=t;until true;else r=t;end else if-4~=a then for n=47,94 do if a<1 then o=e;break;end;y=d;break;end;else y=d;end end else if a<=5 then if 2<a then for e=34,89 do if 4~=a then j=o[y];break;end;k=r[o[s]];break;end;else k=r[o[s]];end else if 5<=a then for e=12,83 do if a<7 then t[j]=k;break;end;a=-2;break;end;else a=-2;end end end a=a+1 end n=n+1;e=l[n];end end end end else t[e[d]]=t[e[f]][e[h]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];t[e[d]]=t[e[f]];n=n+1;e=l[n];t[e[d]]();n=n+1;e=l[n];do return end;end end else if a>5 then if 6>=a then t[e[d]]=t[e[f]]-e[h];else if 6~=a then for n=10,67 do if 8~=a then local s,y,p,a,o,r,l;local n=0;while n>-1 do if n<3 then if 1>n then s=d;y=f;p=h;else if 0<=n then for t=49,82 do if n~=2 then a=e;break;end;o=a[y];break;end;else a=e;end end else if 5>n then if n~=2 then repeat if 4~=n then r=a[s];break;end;l=t[o];for e=1+o,a[p]do l=l..t[e];end;until true;else l=t[o];for e=1+o,a[p]do l=l..t[e];end;end else if 3<=n then repeat if 5<n then n=-2;break;end;t[r]=l;until true;else t[r]=l;end end end n=n+1 end break;end;y[e[f]]=t[e[d]];break;end;else local s,y,p,a,o,r,l;local n=0;while n>-1 do if n<3 then if 1>n then s=d;y=f;p=h;else if 0<=n then for t=49,82 do if n~=2 then a=e;break;end;o=a[y];break;end;else a=e;end end else if 5>n then if n~=2 then repeat if 4~=n then r=a[s];break;end;l=t[o];for e=1+o,a[p]do l=l..t[e];end;until true;else l=t[o];for e=1+o,a[p]do l=l..t[e];end;end else if 3<=n then repeat if 5<n then n=-2;break;end;t[r]=l;until true;else t[r]=l;end end end n=n+1 end end end else if 1<=a then repeat if a<5 then t[e[d]][t[e[f]]]=t[e[h]];break;end;local o;for a=0,6 do if 2>=a then if a>0 then if a>0 then for o=28,82 do if a>1 then t(e[d],e[f]);n=n+1;e=l[n];break;end;t(e[d],e[f]);n=n+1;e=l[n];break;end;else t(e[d],e[f]);n=n+1;e=l[n];end else t(e[d],e[f]);n=n+1;e=l[n];end else if 4>=a then if-1~=a then for o=31,53 do if 3~=a then t(e[d],e[f]);n=n+1;e=l[n];break;end;t(e[d],e[f]);n=n+1;e=l[n];break;end;else t(e[d],e[f]);n=n+1;e=l[n];end else if a~=6 then o=e[d]t[o]=t[o](p(t,o+1,e[f]))n=n+1;e=l[n];else t[e[d]]=t[e[f]];end end end end until true;else local o;for a=0,6 do if 2>=a then if a>0 then if a>0 then for o=28,82 do if a>1 then t(e[d],e[f]);n=n+1;e=l[n];break;end;t(e[d],e[f]);n=n+1;e=l[n];break;end;else t(e[d],e[f]);n=n+1;e=l[n];end else t(e[d],e[f]);n=n+1;e=l[n];end else if 4>=a then if-1~=a then for o=31,53 do if 3~=a then t(e[d],e[f]);n=n+1;e=l[n];break;end;t(e[d],e[f]);n=n+1;e=l[n];break;end;else t(e[d],e[f]);n=n+1;e=l[n];end else if a~=6 then o=e[d]t[o]=t[o](p(t,o+1,e[f]))n=n+1;e=l[n];else t[e[d]]=t[e[f]];end end end end end end end else if 14>a then if 10>=a then if a==10 then for a=0,6 do if a<3 then if 0>=a then t[e[d]]={};n=n+1;e=l[n];else if a~=-3 then repeat if 2~=a then t(e[d],e[f]);n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]];n=n+1;e=l[n];until true;else t(e[d],e[f]);n=n+1;e=l[n];end end else if 5<=a then if a>5 then t(e[d],e[f]);else t(e[d],e[f]);n=n+1;e=l[n];end else if a>-1 then repeat if 3~=a then t(e[d],e[f]);n=n+1;e=l[n];break;end;t(e[d],e[f]);n=n+1;e=l[n];until true;else t(e[d],e[f]);n=n+1;e=l[n];end end end end else local r,b,o,s,u,y,k,j,a;r=e[d];b=t[e[f]];t[r+1]=b;t[r]=b[e[h]];n=n+1;e=l[n];a=0;while a>-1 do if 3<a then if 6<=a then if a>=5 then repeat if 7>a then t[j]=k;break;end;a=-2;until true;else a=-2;end else if 0~=a then for e=23,61 do if a~=4 then j=o[s];break;end;k=y[o[u]];break;end;else k=y[o[u]];end end else if a<=1 then if-3<=a then repeat if 1>a then o=e;break;end;s=d;until true;else o=e;end else if 3~=a then u=f;else y=t;end end end a=a+1 end n=n+1;e=l[n];a=0;while a>-1 do if a>3 then if a>5 then if a<7 then t[j]=k;else a=-2;end else if a>=2 then repeat if 4<a then j=o[s];break;end;k=y[o[u]];until true;else k=y[o[u]];end end else if a<=1 then if-1<=a then repeat if a<1 then o=e;break;end;s=d;until true;else s=d;end else if-2<a then for e=27,92 do if a~=2 then y=t;break;end;u=f;break;end;else y=t;end end end a=a+1 end n=n+1;e=l[n];r=e[d]t[r]=t[r](p(t,r+1,e[f]))n=n+1;e=l[n];t[e[d]][t[e[f]]]=t[e[h]];n=n+1;e=l[n];r=e[d];b=t[e[f]];t[r+1]=b;t[r]=b[e[h]];n=n+1;e=l[n];a=0;while a>-1 do if a>3 then if 5>=a then if 0<=a then repeat if 4~=a then j=o[s];break;end;k=y[o[u]];until true;else j=o[s];end else if 2<=a then repeat if a>6 then a=-2;break;end;t[j]=k;until true;else a=-2;end end else if 1>=a then if a>-1 then repeat if 1>a then o=e;break;end;s=d;until true;else s=d;end else if a==3 then y=t;else u=f;end end end a=a+1 end end else if a<=11 then s[e[f]]=t[e[d]];else if 10<=a then repeat if 12~=a then for a=0,6 do if 3<=a then if 5<=a then if 1~=a then for o=41,68 do if a<6 then t[e[d]]=y[e[f]];n=n+1;e=l[n];break;end;t[e[d]]=y[e[f]];break;end;else t[e[d]]=y[e[f]];end else if a>0 then for o=36,98 do if 4~=a then t[e[d]]=y[e[f]];n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;else t[e[d]]=y[e[f]];n=n+1;e=l[n];end end else if 1>a then t[e[d]]=y[e[f]];n=n+1;e=l[n];else if a~=-1 then repeat if 1<a then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;t[e[d]]=y[e[f]];n=n+1;e=l[n];until true;else t[e[d]]=y[e[f]];n=n+1;e=l[n];end end end end break;end;n=e[f];until true;else for a=0,6 do if 3<=a then if 5<=a then if 1~=a then for o=41,68 do if a<6 then t[e[d]]=y[e[f]];n=n+1;e=l[n];break;end;t[e[d]]=y[e[f]];break;end;else t[e[d]]=y[e[f]];end else if a>0 then for o=36,98 do if 4~=a then t[e[d]]=y[e[f]];n=n+1;e=l[n];break;end;t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;else t[e[d]]=y[e[f]];n=n+1;e=l[n];end end else if 1>a then t[e[d]]=y[e[f]];n=n+1;e=l[n];else if a~=-1 then repeat if 1<a then t[e[d]]=t[e[f]][t[e[h]]];n=n+1;e=l[n];break;end;t[e[d]]=y[e[f]];n=n+1;e=l[n];until true;else t[e[d]]=y[e[f]];n=n+1;e=l[n];end end end end end end end else if 16<=a then if 16<a then if 13<=a then for n=42,73 do if a~=18 then t[e[d]]=t[e[f]]-e[h];break;end;local l=e[d];local d={};for e=1,#k do local e=k[e];for n=0,#e do local e=e[n];local f=e[1];local n=e[2];if f==t and n>=l then d[n]=f[n];e[1]=d;end;end;end;break;end;else local l=e[d];local f={};for e=1,#k do local e=k[e];for n=0,#e do local n=e[n];local d=n[1];local e=n[2];if d==t and e>=l then f[e]=d[e];n[1]=f;end;end;end;end else y[e[f]]=t[e[d]];end else if 12<=a then repeat if a~=14 then for a=0,1 do if a>-1 then repeat if 0<a then if t[e[d]]then n=n+1;else n=e[f];end;break;end;t[e[d]]=s[e[f]];n=n+1;e=l[n];until true;else if t[e[d]]then n=n+1;else n=e[f];end;end end break;end;t[e[d]]=t[e[f]]+t[e[h]];until true;else t[e[d]]=t[e[f]]+t[e[h]];end end end end end end end n=1+n;end;end;return de end;local f=0xff;local h={};local a=(1);local d='';(function(n)local t=n local l=0x00 local e=0x00 t={(function(o)if l>0x2a then return o end l=l+1 e=(e+0xff1-o)%0x18 return(e%0x03==0x1 and(function(t)if not n[t]then e=e+0x01 n[t]=(0x7);d={d..'\58 a',d};h[a]=te();a=a+(1);d[1]='\58'..d[1];f[2]=0xff;end return true end)'dVKwD'and t[0x3](0x1e0+o))or(e%0x03==0x0 and(function(t)if not n[t]then e=e+0x01 n[t]=(0x5c);end return true end)'SFMam'and t[0x2](o+0x3bc))or(e%0x03==0x2 and(function(t)if not n[t]then e=e+0x01 n[t]=(0x9d);d='\37';f={function()f()end};d=d..'\100\43';end return true end)'FuorP'and t[0x1](o+0xb2))or o end),(function(d)if l>0x23 then return d end l=l+1 e=(e+0x5f9-d)%0xc return(e%0x03==0x0 and(function(t)if not n[t]then e=e+0x01 n[t]=(0xe1);end return true end)'Zt_SS'and t[0x1](0x380+d))or(e%0x03==0x1 and(function(t)if not n[t]then e=e+0x01 n[t]=(0x8e);end return true end)'HWJkY'and t[0x2](d+0x19b))or(e%0x03==0x2 and(function(t)if not n[t]then e=e+0x01 n[t]=(0x75);end return true end)'jIkpZ'and t[0x3](d+0x87))or d end),(function(o)if l>0x29 then return o end l=l+1 e=(e+0xce5-o)%0x34 return(e%0x03==0x1 and(function(t)if not n[t]then e=e+0x01 n[t]=(0x7d);f[2]=(f[2]*(de(function()h()end,p(d))-de(f[1],p(d))))+1;h[a]={};f=f[2];a=a+f;end return true end)'dVdnS'and t[0x3](0x1e3+o))or(e%0x03==0x2 and(function(t)if not n[t]then e=e+0x01 n[t]=(0x6a);end return true end)'gPGoL'and t[0x1](o+0x32d))or(e%0x03==0x0 and(function(t)if not n[t]then e=e+0x01 n[t]=(0xe0);h[a]=fe();a=a+f;end return true end)'PVfoV'and t[0x2](o+0x14e))or o end)}t[0x1](0x19a6)end){};local e=c(p(h));return e(...);end return ne((function()local n={}local e=0x01;local t;if o.ClqRmYwJ then t=o.ClqRmYwJ(ne)else t=''end if o.ptXCgLvz(t,o.eFkjdYUI)then e=e+0;else e=e+1;end n[e]=0x02;n[n[e]+0x01]=0x03;return n;end)(),...)end)((function(n,e,t,d,f,l)local l;if n>=4 then if 5<n then if 6>=n then do return f[t]end;else if 3<=n then repeat if 8~=n then do return setmetatable({},{['__\99\97\108\108']=function(e,f,d,t,n)if n then return e[n]elseif t then return e else e[f]=d end end})end break;end;do return t(n,nil,t);end until true;else do return setmetatable({},{['__\99\97\108\108']=function(e,f,d,t,n)if n then return e[n]elseif t then return e else e[f]=d end end})end end end else if n>4 then local n=d;do return function()local e=e(t,n(n,n),n(n,n));n(1);return e;end;end;else local n=d;local d,f,l=f(2);do return function()local a,o,e,t=e(t,n(n,n),n(n,n)+3);n(4);return(t*d)+(e*f)+(o*l)+a;end;end;end end else if 2<=n then if n>=-1 then for l=22,68 do if 3>n then do return 16777216,65536,256 end;break;end;do return e(1),e(4,f,d,t,e),e(5,f,d,t)end;break;end;else do return e(1),e(4,f,d,t,e),e(5,f,d,t)end;end else if-2<=n then for l=12,54 do if n<1 then do return e(1),e(4,f,d,t,e),e(5,f,d,t)end;break;end;do return function(n,e,t)if t then local e=(n/2^(e-1))%2^((t-1)-(e-1)+1);return e-e%1;else local e=2^(e-1);return(n%(e+e)>=e)and 1 or 0;end;end;end;break;end;else do return function(t,e,n)if n then local e=(t/2^(e-1))%2^((n-1)-(e-1)+1);return e-e%1;else local e=2^(e-1);return(t%(e+e)>=e)and 1 or 0;end;end;end;end end end end),...)
