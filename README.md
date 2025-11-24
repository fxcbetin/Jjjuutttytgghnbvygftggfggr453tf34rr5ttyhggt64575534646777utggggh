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
