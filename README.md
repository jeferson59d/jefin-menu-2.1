-- ===== SERVIÇOS =====
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- ===== VARIÁVEIS =====
local voando=false; local velocidadeVoo=50; local vooConn
local noclip=false; local invisivel=false; local deus=false
local ESPAtivo=false; local linhaESPAtiva=false; local aimbotAtivo=false
local ESPVidaAtivo=false
local FOV=100
local Personagem = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HRP = Personagem:WaitForChild("HumanoidRootPart")
local Humanoid = Personagem:WaitForChild("Humanoid")
local ESPBoxes={}; local ESPLines={}; local ESPVidaLabels={}

-- ===== KEYS =====
local keysValidas = { ["TEMP1MIN"]=60, ["TEMP10MIN"]=600, ["VITALICIA"]=0 }
local keysUsadas = {}
local keyAtiva=false; local tempoExpira=0

-- ===== FUNÇÕES AUXILIARES =====
local function criarFrame(parent,size,pos,color)
    local f=Instance.new("Frame",parent)
    f.Size=size; f.Position=pos; f.BackgroundColor3=color; f.BorderSizePixel=0
    return f
end
local function criarTexto(parent,text,pos,size,color,font,txtSize)
    local t=Instance.new("TextLabel",parent)
    t.Text=text; t.Size=size; t.Position=pos; t.BackgroundTransparency=1; t.TextColor3=color
    t.Font=font; t.TextSize=txtSize
    return t
end
local function criarBotao(parent,text,pos,size,bgColor,txtColor,txtSize)
    local b=Instance.new("TextButton",parent)
    b.Text=text; b.Size=size; b.Position=pos
    b.BackgroundColor3=bgColor; b.TextColor3=txtColor
    b.Font=Enum.Font.SourceSansBold; b.TextSize=txtSize; b.BorderSizePixel=0
    return b
end

-- ===== TELA DE KEY =====
local function abrirTelaKey(msg)
    local tela = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
    tela.Name = "TelaChave"; tela.ResetOnSpawn = false; tela.Enabled = true
    local frame = criarFrame(tela, UDim2.new(0,280,0,140), UDim2.new(0.35,0,0.3,0), Color3.fromRGB(25,25,25))
    local label = criarTexto(frame, msg or "Digite a Key:", UDim2.new(0,0,0,10), UDim2.new(1,0,0,30), Color3.fromRGB(255,255,255), Enum.Font.SourceSansBold, 18)
    local caixa = Instance.new("TextBox", frame)
    caixa.Size, caixa.Position = UDim2.new(0.8,0,0,40), UDim2.new(0.1,0,0,50)
    caixa.BackgroundColor3, caixa.TextColor3 = Color3.fromRGB(45,45,45), Color3.new(1,1,1)
    caixa.ClearTextOnFocus, caixa.Font, caixa.TextSize = false, Enum.Font.SourceSansBold, 16
    local botao = criarBotao(frame,"Confirmar",UDim2.new(0.25,0,0,100),UDim2.new(0.5,0,0,30),Color3.fromRGB(70,70,70),Color3.fromRGB(255,255,255),16)

    botao.MouseButton1Click:Connect(function()
        local key = caixa.Text
        if keysUsadas[key] then
            label.Text = "Key já usada!"; caixa.Text=""; return
        end
        local tempo = keysValidas[key]
        if tempo ~= nil then
            keyAtiva = true
            tempoExpira = tempo>0 and tick()+tempo or 0
            if tempo>0 then keysUsadas[key]=true end
            tela:Destroy()
            gui.Enabled = true
        else
            label.Text="Key incorreta!"; caixa.Text=""
        end
    end)
    return tela
end

local telaChaveInicial = abrirTelaKey()

-- ===== MENU PRINCIPAL =====
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name="MenuJefin"; gui.ResetOnSpawn=false; gui.Enabled=false
local frameMenu = criarFrame(gui,UDim2.new(0,220,0,350),UDim2.new(0.35,0,0.25,0),Color3.fromRGB(30,30,30))

-- ===== MENU ARRASTÁVEL =====
local arrastando=false; local dragInput; local mousePos; local framePos
local function atualizar(input)
    local delta=input.Position-mousePos
    frameMenu.Position=UDim2.new(framePos.X.Scale, framePos.X.Offset+delta.X, framePos.Y.Scale, framePos.Y.Offset+delta.Y)
end
frameMenu.InputBegan:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.MouseButton1 then
        arrastando=true; mousePos=input.Position; framePos=frameMenu.Position
        input.Changed:Connect(function()
            if input.UserInputState==Enum.UserInputState.End then arrastando=false end
        end)
    end
end)
frameMenu.InputChanged:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.MouseMovement then dragInput=input end
end)
UserInputService.InputChanged:Connect(function(input)
    if input==dragInput and arrastando then atualizar(input) end
end)

-- ===== ABAS HORIZONTAIS =====
local abaFrame = criarFrame(frameMenu,UDim2.new(1,0,0,25),UDim2.new(0,0,0,0),Color3.fromRGB(50,50,50))
local abas={"Player","Combat","Teleport","Key"}; local abaAtiva="Player"
local conteudoAbas={}
for i,nome in ipairs(abas) do
    local b = criarBotao(abaFrame,nome,UDim2.new(0.05+(i-1)*0.23,0,0,0),UDim2.new(0.2,0,1,0),Color3.fromRGB(70,70,70),Color3.fromRGB(255,255,255),14)
    b.MouseButton1Click:Connect(function()
        abaAtiva=nome
        for k,v in pairs(conteudoAbas) do v.Visible = k==abaAtiva end
    end)
end
for _,nome in ipairs(abas) do
    local f = criarFrame(frameMenu,UDim2.new(1,0,1,-25),UDim2.new(0,0,0,25),Color3.fromRGB(30,30,30))
    f.Visible = nome=="Player"
    conteudoAbas[nome]=f
end

-- ===== ABA PLAYER =====
local fPlayer = conteudoAbas["Player"]
local botaoFly = criarBotao(fPlayer,"Ativar Fly",UDim2.new(0.05,0,0,10),UDim2.new(0.9,0,0,30),Color3.fromRGB(60,60,60),Color3.new(1,1,1),16)
local botaoNoclip = criarBotao(fPlayer,"Ativar NoClip",UDim2.new(0.05,0,0,50),UDim2.new(0.9,0,0,30),Color3.fromRGB(60,60,60),Color3.new(1,1,1),16)
local botaoInvisivel = criarBotao(fPlayer,"Ativar Invisibilidade",UDim2.new(0.05,0,0,90),UDim2.new(0.9,0,0,30),Color3.fromRGB(60,60,60),Color3.new(1,1,1),16)
local botaoGod = criarBotao(fPlayer,"Ativar GodMode",UDim2.new(0.05,0,0,130),UDim2.new(0.9,0,0,30),Color3.fromRGB(60,60,60),Color3.new(1,1,1),16)

-- Slider Fly
local sliderFrame = criarFrame(fPlayer,UDim2.new(0.9,0,0,25),UDim2.new(0.05,0,0,170),Color3.fromRGB(45,45,45))
local sliderBar = criarFrame(sliderFrame,UDim2.new(1,-20,0,10),UDim2.new(0,10,0.5,-5),Color3.fromRGB(70,70,70))
local sliderButton = Instance.new("TextButton", sliderBar)
sliderButton.Size = UDim2.new(0,16,0,16); sliderButton.Position = UDim2.new(0,0,0.5,-8)
sliderButton.BackgroundColor3 = Color3.fromRGB(150,150,150); sliderButton.Text = ""
local labelVelocidade = criarTexto(sliderFrame,"Velocidade: 50",UDim2.new(0,0,1,0),UDim2.new(1,0,0,18),Color3.new(1,1,1),Enum.Font.SourceSansBold,14)

-- ===== ABA COMBAT =====
local fCombat = conteudoAbas["Combat"]
local botaoESP = criarBotao(fCombat,"Ativar ESP Box",UDim2.new(0.05,0,0,10),UDim2.new(0.9,0,0,30),Color3.fromRGB(60,60,60),Color3.new(1,1,1),16)
local botaoLinhaESP = criarBotao(fCombat,"Ativar ESP Line",UDim2.new(0.05,0,0,50),UDim2.new(0.9,0,0,30),Color3.fromRGB(60,60,60),Color3.new(1,1,1),16)
local botaoAimbot = criarBotao(fCombat,"Ativar Aimbot",UDim2.new(0.05,0,0,90),UDim2.new(0.9,0,0,30),Color3.fromRGB(60,60,60),Color3.new(1,1,1),16)
local botaoESPVida = criarBotao(fCombat,"Ativar ESP Vida",UDim2.new(0.05,0,0,130),UDim2.new(0.9,0,0,30),Color3.fromRGB(60,60,60),Color3.new(1,1,1),16)

-- Slider FOV
local sliderFOVFrame = criarFrame(fCombat,UDim2.new(0.9,0,0,25),UDim2.new(0.05,0,0,170),Color3.fromRGB(45,45,45))
local sliderFOVBar = criarFrame(sliderFOVFrame,UDim2.new(1,-20,0,10),UDim2.new(0,10,0.5,-5),Color3.fromRGB(70,70,70))
local sliderFOVButton = Instance.new("TextButton", sliderFOVBar)
sliderFOVButton.Size = UDim2.new(0,16,0,16); sliderFOVButton.Position = UDim2.new(0,0,0.5,-8)
sliderFOVButton.BackgroundColor3 = Color3.fromRGB(150,150,150); sliderFOVButton.Text = ""
local labelFOV = criarTexto(sliderFOVFrame,"FOV: 100",UDim2.new(0,0,1,0),UDim2.new(1,0,0,18),Color3.new(1,1,1),Enum.Font.SourceSansBold,14)

-- ===== ABA TELEPORT =====
local fTeleport = conteudoAbas["Teleport"]
local botaoTeleporte = criarBotao(fTeleport,"Abrir Lista",UDim2.new(0.05,0,0,10),UDim2.new(0.9,0,0,30),Color3.fromRGB(60,60,60),Color3.new(1,1,1),16)
local listaPlayers = criarFrame(fTeleport,UDim2.new(0.9,0,0,200),UDim2.new(0.05,0,0,50),Color3.fromRGB(50,50,50))
listaPlayers.Visible = false
local pesquisaBox = Instance.new("TextBox", fTeleport)
pesquisaBox.Size = UDim2.new(0.9,0,0,25); pesquisaBox.Position = UDim2.new(0.05,0,0,260)
pesquisaBox.BackgroundColor3 = Color3.fromRGB(45,45,45); pesquisaBox.TextColor3 = Color3.new(1,1,1)
pesquisaBox.PlaceholderText = "Pesquisar jogador..."; pesquisaBox.Font = Enum.Font.SourceSansBold; pesquisaBox.TextSize = 16

-- ===== ABA KEY =====
local fKey = conteudoAbas["Key"]
local tempoRestanteKey = criarTexto(fKey,"",UDim2.new(0,0,0,20),UDim2.new(1,0,0,40),Color3.fromRGB(255,255,0),Enum.Font.SourceSansBold,16)

-- ===== TOGGLE MENU H =====
UserInputService.InputBegan:Connect(function(input,gp)
    if gp then return end
    if input.KeyCode==Enum.KeyCode.H and keyAtiva then
        gui.Enabled = not gui.Enabled
    end
end)

-- ===== CHECAGEM DE EXPIRAÇÃO =====
RunService.RenderStepped:Connect(function()
    if keyAtiva then
        if tempoExpira>0 then
            local tRest = math.max(0,math.floor(tempoExpira-tick()))
            tempoRestanteKey.Text = "Tempo restante da key: "..tRest.."s"
            if tRest<=0 then
                keyAtiva=false; gui.Enabled=false
                abrirTelaKey("Key expirou! Insira uma nova key:")
            end
        else
            tempoRestanteKey.Text = "Key Vitalícia ativa"
        end
    else
        tempoRestanteKey.Text = "Nenhuma key ativa"
    end
end)

-- ===== FLY =====
local arrastandoSlider=false
sliderButton.MouseButton1Down:Connect(function() arrastandoSlider=true end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.MouseButton1 then arrastandoSlider=false end
end)
RunService.RenderStepped:Connect(function()
    if arrastandoSlider then
        local mx = UserInputService:GetMouseLocation().X
        local pct = math.clamp((mx - sliderBar.AbsolutePosition.X)/sliderBar.AbsoluteSize.X,0,1)
        sliderButton.Position = UDim2.new(pct,-8,0.5,-8)
        velocidadeVoo = math.floor(20 + pct*180)
        labelVelocidade.Text = "Velocidade: "..velocidadeVoo
    end
end)
local function iniciarFly()
    if voando then return end; voando=true; botaoFly.Text="Desativar Fly"
    vooConn = RunService.RenderStepped:Connect(function()
        local dir=Vector3.new()
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir=dir+Camera.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir=dir-Camera.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir=dir-CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir=dir+Camera.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir=dir+Vector3.new(0,1,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then dir=dir-Vector3.new(0,1,0) end
        if dir.Magnitude>0 then HRP.Velocity=dir.Unit*velocidadeVoo else HRP.Velocity=Vector3.new(0,0,0) end
    end)
end
local function pararFly() voando=false; botaoFly.Text="Ativar Fly"; if vooConn then vooConn:Disconnect() end; HRP.Velocity=Vector3.new(0,0,0) end
botaoFly.MouseButton1Click:Connect(function() if voando then pararFly() else iniciarFly() end end)

-- ===== NOCLIP =====
botaoNoclip.MouseButton1Click:Connect(function()
    noclip = not noclip
    botaoNoclip.Text = noclip and "Desativar NoClip" or "Ativar NoClip"
end)
RunService.RenderStepped:Connect(function()
    if noclip then
        for _, part in pairs(Personagem:GetChildren()) do
            if part:IsA("BasePart") then part.CanCollide=false end
        end
    end
end)

-- ===== INVISIBILIDADE =====
botaoInvisivel.MouseButton1Click:Connect(function()
    invisivel = not invisivel
    for _, part in pairs(Personagem:GetChildren()) do
        if part:IsA("BasePart") then part.Transparency=invisivel and 1 or 0
        elseif part:IsA("Decal") then part.Transparency=invisivel and 1 or 0
        elseif part:IsA("ParticleEmitter") then part.Enabled = not invisivel end
    end
    botaoInvisivel.Text = invisivel and "Desativar Invisibilidade" or "Ativar Invisibilidade"
end)

-- ===== GOD MODE =====
botaoGod.MouseButton1Click:Connect(function()
    deus = not deus
    Humanoid.MaxHealth = deus and math.huge or 100
    Humanoid.Health = deus and math.huge or Humanoid.Health
    botaoGod.Text = deus and "Desativar GodMode" or "Ativar GodMode"
end)

-- ===== ESP / AIMBOT =====
botaoESP.MouseButton1Click:Connect(function() ESPAtivo = not ESPAtivo; botaoESP.Text=ESPAtivo and "Desativar ESP Box" or "Ativar ESP Box" end)
botaoLinhaESP.MouseButton1Click:Connect(function() linhaESPAtiva = not linhaESPAtiva; botaoLinhaESP.Text=linhaESPAtiva and "Desativar ESP Line" or "Ativar ESP Line" end)
botaoESPVida.MouseButton1Click:Connect(function() ESPVidaAtivo = not ESPVidaAtivo; botaoESPVida.Text=ESPVidaAtivo and "Desativar ESP Vida" or "Ativar ESP Vida" end)
botaoAimbot.MouseButton1Click:Connect(function() aimbotAtivo = not aimbotAtivo; botaoAimbot.Text=aimbotAtivo and "Desativar Aimbot" or "Ativar Aimbot" end)

-- ===== SLIDER FOV =====
local arrastandoFOV=false
sliderFOVButton.MouseButton1Down:Connect(function() arrastandoFOV=true end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.MouseButton1 then arrastandoFOV=false end
end)
RunService.RenderStepped:Connect(function()
    if arrastandoFOV then
        local mx = UserInputService:GetMouseLocation().X
        local pct = math.clamp((mx - sliderFOVBar.AbsolutePosition.X)/sliderFOVBar.AbsoluteSize.X,0,1)
        sliderFOVButton.Position = UDim2.new(pct,-8,0.5,-8)
        FOV = math.floor(30 + pct*170)
        labelFOV.Text = "FOV: "..FOV
    end
end)

-- ===== ESP RENDER =====
RunService.RenderStepped:Connect(function()
    if not keyAtiva then return end
    for _,plr in pairs(Players:GetPlayers()) do
        if plr~=LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = plr.Character.HumanoidRootPart
            local pos,vis = Camera:WorldToViewportPoint(hrp.Position)
            -- ESP BOX
            if ESPAtivo then
                if not ESPBoxes[plr] then
                    local box = Instance.new("Frame", gui); box.Size=UDim2.new(0,50,0,50)
                    box.BackgroundTransparency=0.6; box.BackgroundColor3=Color3.fromRGB(0,255,0); box.BorderSizePixel=0
                    ESPBoxes[plr]=box
                end
                ESPBoxes[plr].Position=UDim2.new(0,pos.X-25,0,pos.Y-25); ESPBoxes[plr].Visible=vis
            elseif ESPBoxes[plr] then ESPBoxes[plr].Visible=false end
            -- ESP LINE
            if linhaESPAtiva then
                if not ESPLines[plr] then
                    local line = Instance.new("Frame", gui); line.Size=UDim2.new(0,2,0,2)
                    line.BorderSizePixel=0; line.BackgroundColor3=Color3.fromRGB(255,255,255)
                    ESPLines[plr]=line
                end
                local startPos = Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y)
                local endPos = Vector2.new(pos.X,pos.Y)
                local delta = endPos - startPos
                ESPLines[plr].Size=UDim2.new(0,2,0,delta.Magnitude); ESPLines[plr].Position=UDim2.new(0,startPos.X,0,startPos.Y)
                ESPLines[plr].Rotation=math.deg(math.atan2(delta.Y,delta.X))+90; ESPLines[plr].Visible=vis
            elseif ESPLines[plr] then ESPLines[plr].Visible=false end
            -- ESP VIDA
            if ESPVidaAtivo then
                if not ESPVidaLabels[plr] then
                    local lbl=Instance.new("TextLabel", gui)
                    lbl.Size=UDim2.new(0,50,0,14); lbl.BackgroundTransparency=1; lbl.TextColor3=Color3.fromRGB(255,0,0)
                    lbl.Font=Enum.Font.SourceSansBold; lbl.TextSize=14; ESPVidaLabels[plr]=lbl
                end
                local humanoid = plr.Character:FindFirstChild("Humanoid")
                ESPVidaLabels[plr].Text = humanoid and math.floor(humanoid.Health) or "0"
                ESPVidaLabels[plr].Position=UDim2.new(0,pos.X-25,0,pos.Y-35); ESPVidaLabels[plr].Visible=vis
            elseif ESPVidaLabels[plr] then ESPVidaLabels[plr].Visible=false end
        end
    end
    -- AIMBOT
    if aimbotAtivo then
        local alvoMaisProximo,menorDist=nil,math.huge
        local mousePos = UserInputService:GetMouseLocation()
        for _,plr in pairs(Players:GetPlayers()) do
            if plr~=LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local pos,vis = Camera:WorldToViewportPoint(plr.Character.HumanoidRootPart.Position)
                local dist = (Vector2.new(pos.X,pos.Y)-Vector2.new(mousePos.X,mousePos.Y)).Magnitude
                if dist<menorDist then menorDist=dist; alvoMaisProximo=plr end
            end
        end
        if alvoMaisProximo and alvoMaisProximo.Character then
            local alvoPos = alvoMaisProximo.Character:FindFirstChild("Head") and alvoMaisProximo.Character.Head.Position or alvoMaisProximo.Character.HumanoidRootPart.Position
            Camera.CFrame=CFrame.lookAt(Camera.CFrame.Position,alvoPos)
        end
    end
end)
