--[[
    CYBERNODRY - LOGIC STRUCTURE CORRECTOR (V22 - BRAIN RESTORED)
    - FIX: Interrogação obrigatória em perguntas.
    - FIX: Vírgula correta (Senhor, / O senhor,).
    - FIX: Acentos (Olá / Amanhã / Você).
    - MANTIDO: Botão [X] para fechar.
    - By: CyberNoDry
]]

local TextChatService = game:GetService("TextChatService")
local CoreGui = game:GetService("CoreGui")

local function LogicFix(text)
    if text == "" then return "" end
    
    -- 1. IDENTIFICA SE É PERGUNTA (Pelo início ou se já tiver ?)
    local lowerRaw = text:lower()
    local ehPergunta = text:find("%?") ~= nil
    local starts = {"vai", "pode", "quer", "tem", "será", "está", "como", "qual", "quem", "onde"}
    
    for _, s in pairs(starts) do
        if lowerRaw:find("^" .. s) or lowerRaw:find("^o%s+senhor%s+" .. s) or lowerRaw:find("^senhor%s+" .. s) then
            ehPergunta = true
            break
        end
    end

    -- 2. DICIONÁRIO DE CORREÇÃO (Sem apagar nada!)
    local correcoes = {
        ["treinamento"] = {"terimantento", "treinamenrto", "treinameto", "trerimamento"},
        ["recrutamento"] = {"recrutameto", "recutamento"},
        ["amanhã"] = {"amanha"},
        ["olá"] = {"ola"},
        ["você"] = {"vc", "voce"},
        ["não"] = {"nao"},
        ["hoje"] = {"hj"}
    }

    local words = {}
    for word in text:gmatch("%S+") do
        local clean = word:lower():gsub("[%p]", "")
        local found = false
        for correto, erros in pairs(correcoes) do
            if clean == correto:lower() or table.find(erros, clean) then 
                word = correto
                found = true 
                break 
            end
        end
        if not found then
            local b = {["e"]="é", ["esta"]="está", ["pq"]="porque", ["tbm"]="também", ["so"]="só"}
            if b[clean] then word = b[clean] end
        end
        table.insert(words, word)
    end
    text = table.concat(words, " ")

    -- 3. AJUSTE DE VÍRGULAS (PADRÃO EB)
    local patentes = {"senhor", "senhora", "sargento", "tenente", "capitão", "cabo", "recruta"}
    for _, v in pairs(patentes) do
        -- Transforma "Senhor você" em "Senhor, você"
        text = text:gsub("^([Ss]enhor)%s", "%1, ")
        -- Transforma "O senhor vai" em "O senhor, vai"
        text = text:gsub("([Oo]%s[Ss]enhor)%s", "%1, ")
        -- "hoje senhor" em "hoje, senhor"
        if text:lower():match("%s" .. v .. "$") then
            text = text:gsub("%s(" .. v .. ")$", ", %1")
        end
    end

    -- 4. FINALIZAÇÃO DA PONTUAÇÃO (TRAVA)
    text = text:sub(1,1):upper() .. text:sub(2)
    text = text:gsub("[%?%.%!%s]+$", "") -- Limpa o fim

    if ehPergunta then
        text = text .. "?"
    else
        text = text .. "."
    end
    
    return text
end

local function Create(cls, props)
    local i = Instance.new(cls)
    for k, v in pairs(props) do i[k] = v end
    return i
end

if CoreGui:FindFirstChild("CyberLogicChat") then CoreGui.CyberLogicChat:Destroy() end
local SG = Create("ScreenGui", { Name = "CyberLogicChat", Parent = CoreGui, ResetOnSpawn = false })
local Main = Create("Frame", { Parent = SG, BackgroundColor3 = Color3.fromRGB(15, 15, 15), Position = UDim2.new(0.5, -150, 0.7, 0), Size = UDim2.new(0, 300, 0, 85), Active = true, Draggable = true })
Create("UIStroke", { Parent = Main, Thickness = 2, Color = Color3.fromRGB(255, 215, 0) })
Create("UICorner", { Parent = Main })

local CloseBtn = Create("TextButton", { Parent = Main, Text = "X", Position = UDim2.new(1, -25, 0, 5), Size = UDim2.new(0, 20, 0, 20), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(255, 50, 50), Font = Enum.Font.GothamBold, TextSize = 16 })
CloseBtn.MouseButton1Click:Connect(function() SG:Destroy() end)

local Inp = Create("TextBox", { Parent = Main, PlaceholderText = "Pergunta ou afirmação...", Position = UDim2.new(0.05, 0, 0.45, 0), Size = UDim2.new(0.9, 0, 0, 30), BackgroundColor3 = Color3.fromRGB(25, 25, 25), TextColor3 = Color3.new(1, 1, 1), Font = Enum.Font.Gotham, TextSize = 14, ClearTextOnFocus = false })
Create("UICorner", { Parent = Inp })

Inp.FocusLost:Connect(function(enter)
    if enter and Inp.Text ~= "" then
        local res = LogicFix(Inp.Text)
        if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
            local channel = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
            if channel then channel:SendAsync(res) end
        else
            game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(res, "All")
        end
        Inp.Text = ""
    end
end)
