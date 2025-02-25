local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = game.Workspace.CurrentCamera
local UIS = game:GetService("UserInputService")
local Mouse = LocalPlayer:GetMouse()

local AimbotEnabled = false
local AimlockEnabled = false
local NoRecoilEnabled = false

-- Intensidades ajustáveis (0 a 100%)
local AimbotIntensity = 90
local AimlockIntensity = 80
local NoRecoilIntensity = 75

-- Nome exato da arma que queremos ativar o aimbot
local ArmaPermitida = "Arisaka 7.7mm"

-- Criar a GUI com bolinhas flutuantes para controlar as intensidades
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)

-- Função para criar bolinhas flutuantes
local function CreateControlBall(name, position)
    local Ball = Instance.new("Frame", ScreenGui)
    Ball.Name = name
    Ball.Size = UDim2.new(0, 40, 0, 40)
    Ball.Position = position
    Ball.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    Ball.AnchorPoint = Vector2.new(0.5, 0.5)
    Ball.BorderRadius = UDim.new(0, 20)
    Ball.Active = true
    Ball.Draggable = true

    local Label = Instance.new("TextLabel", Ball)
    Label.Size = UDim2.new(1, 0, 0, 20)
    Label.Position = UDim2.new(0, 0, 1, 0)
    Label.Text = "0%"
    Label.BackgroundTransparency = 1
    Label.TextColor3 = Color3.fromRGB(255, 255, 255)
    Label.TextScaled = true

    return Ball, Label
end

-- Criar bolinhas de controle para Aimbot, Aimlock, NoRecoil
local AimbotBall, AimbotLabel = CreateControlBall("AimbotBall", UDim2.new(0.1, 0, 0.1, 0))
local AimlockBall, AimlockLabel = CreateControlBall("AimlockBall", UDim2.new(0.3, 0, 0.1, 0))
local NoRecoilBall, NoRecoilLabel = CreateControlBall("NoRecoilBall", UDim2.new(0.5, 0, 0.1, 0))

-- Atualizar intensidade e ativar/desativar as opções
local function UpdateIntensity()
    AimbotIntensity = AimbotBall.Position.X.Scale * 100
    AimlockIntensity = AimlockBall.Position.X.Scale * 100
    NoRecoilIntensity = NoRecoilBall.Position.X.Scale * 100

    -- Atualiza os labels com as intensidades
    AimbotLabel.Text = math.floor(AimbotIntensity) .. "%"
    AimlockLabel.Text = math.floor(AimlockIntensity) .. "%"
    NoRecoilLabel.Text = math.floor(NoRecoilIntensity) .. "%"

    -- Ativar/Desativar as opções com base na intensidade
    AimbotEnabled = AimbotIntensity > 0
    AimlockEnabled = AimlockIntensity > 0
    NoRecoilEnabled = NoRecoilIntensity > 0
end

-- Função para verificar se o jogador está com a arma certa
local function TemArmaCerta()
    local Character = LocalPlayer.Character
    if Character and Character:FindFirstChildOfClass("Tool") then
        local Arma = Character:FindFirstChildOfClass("Tool")
        return Arma.Name == ArmaPermitida
    end
    return false
end

-- Função para encontrar o inimigo
local function GetCurrentTarget()
    local ShortestDistance = math.huge
    local MelhorAlvo = nil

    for _, Player in pairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Head") then
            if Player.TeamColor == BrickColor.new("Bright red") then
                local Head = Player.Character.Head
                local DistanceFromPlayer = (Head.Position - LocalPlayer.Character.Head.Position).Magnitude
                
                -- Definir a meia distância (máximo de 150 studs)
                if DistanceFromPlayer < 150 then
                    local EnemyPos, OnScreen = Camera:WorldToViewportPoint(Head.Position)
                    if OnScreen then
                        local DistanceToCenter = (Vector2.new(EnemyPos.X, EnemyPos.Y) - UserInputService:GetMouseLocation()).Magnitude
                        if DistanceToCenter < ShortestDistance and DistanceToCenter < 100 then
                            ShortestDistance = DistanceToCenter
                            MelhorAlvo = Head
                        end
                    end
                end
            end
        end
    end
    return MelhorAlvo
end

-- Função para suavizar a mira
local function SmoothAim(Target, Speed)
    if Target then
        local TargetPos = Target.Position
        local CurrentCFrame = Camera.CFrame
        local NewDirection = (TargetPos - CurrentCFrame.Position).unit

        -- Suaviza a transição para a mira
        local SmoothedCFrame = CFrame.new(CurrentCFrame.Position, CurrentCFrame.Position + (CurrentCFrame.LookVector:Lerp(NewDirection, Speed)))
        Camera.CFrame = SmoothedCFrame
    end
end

-- Função para aumentar a precisão no tiro
local function AimbotShoot(Target)
    if Target then
        SmoothAim(Target, AimbotIntensity / 100) -- Aplica a intensidade do aimbot
    end
end

-- Função de Aimlock
local function Aimlock(Target)
    if Target then
        SmoothAim(Target, AimlockIntensity / 100) -- Aplica a intensidade do aimlock
    end
end

-- Função de NoRecoil
local function NoRecoil()
    -- Aqui você pode adicionar a lógica para remover o recoil da arma.
    -- A intensidade pode ser aplicada para reduzir ou eliminar o recoil.
    if NoRecoilIntensity > 0 then
        -- Modifique a lógica de recoil com base na intensidade
    end
end

-- Ativar mira ao atirar, mas só se estiver com o Arisaka e mirando o mesmo alvo
Mouse.Button1Down:Connect(function()
    if AimbotEnabled and TemArmaCerta() then
        local SelectedTarget = GetCurrentTarget() -- Escolher o inimigo que estamos mirando no momento
        if SelectedTarget then
            AimbotShoot(SelectedTarget) -- Puxa a mira para a cabeça do inimigo com a intensidade do aimbot
        end
    end
end)

-- Função principal do Aimbot, Aimlock e NoRecoil
local function Aimbot()
    while true do
        if AimbotEnabled and TemArmaCerta() then
            local SelectedTarget = GetCurrentTarget()
            if SelectedTarget then
                -- Seguir o alvo com a intensidade configurada de Aimbot e Aimlock
                SmoothAim(SelectedTarget, AimbotIntensity / 100) -- Aimbot
                Aimlock(SelectedTarget) -- Aimlock
                NoRecoil() -- NoRecoil
            else
                SelectedTarget = nil
            end
        end
        task.wait(0.01)  
    end
end

-- Iniciar Aimbot
task.spawn(Aimbot)

-- Atualizar intensidade com base na posição das bolinhas
AimbotBall.Changed:Connect(function()
    UpdateIntensity()
end)
AimlockBall.Changed:Connect(function()
    UpdateIntensity()
end)
NoRecoilBall.Changed:Connect(function()
    UpdateIntensity()
end)
