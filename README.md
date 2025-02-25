local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Criar GUI do Menu
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local MenuFrame = Instance.new("Frame", ScreenGui)
local MinimizeButton = Instance.new("TextButton", MenuFrame)
local MaximizeButton = Instance.new("TextButton", MenuFrame)
local CreditsLabel = Instance.new("TextLabel", MenuFrame)

-- Estilização do Menu (Forma Quadrada e Azul)
MenuFrame.Size = UDim2.new(0, 400, 0, 400)
MenuFrame.Position = UDim2.new(0.5, -200, 0.5, -200)
MenuFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 255) -- Azul
MenuFrame.Visible = true

-- Botão para minimizar
MinimizeButton.Size = UDim2.new(0, 100, 0, 50)
MinimizeButton.Position = UDim2.new(0.5, -150, 0, 0)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.Text = "Minimizar"

-- Botão para maximizar
MaximizeButton.Size = UDim2.new(0, 100, 0, 50)
MaximizeButton.Position = UDim2.new(0.5, 50, 0, 0)
MaximizeButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
MaximizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MaximizeButton.Text = "Maximizar"
MaximizeButton.Visible = false

-- Label de créditos
CreditsLabel.Size = UDim2.new(1, 0, 0, 50)
CreditsLabel.Position = UDim2.new(0, 0, 0.9, 0)
CreditsLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 255) -- Azul
CreditsLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
CreditsLabel.Text = "Creditos: SACOLA e THVEZEIRA"
CreditsLabel.TextScaled = true

-- Funções do Menu
local autoRevistarEnabled = false
local hitboxEnabled = false
local flyEnabled = false
local autoCLEnabled = false

-- Função para ativar/desativar Auto Revistar
local function toggleAutoRevistar()
    autoRevistarEnabled = not autoRevistarEnabled
end

-- Função para ativar/desativar Hitbox
local function toggleHitbox()
    hitboxEnabled = not hitboxEnabled
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if hitboxEnabled then
                local hitBox = Instance.new("Part")
                hitBox.Size = Vector3.new(5, 5, 5) -- Tamanho da hitbox
                hitBox.Position = player.Character.HumanoidRootPart.Position
                hitBox.Color = Color3.new(1, 0, 0) -- Vermelho
                hitBox.Transparency = 0.5
                hitBox.Anchored = true
                hitBox.CanCollide = false
                hitBox.Parent = workspace

                -- Conectar dano ao hitbox
                hitBox.Touched:Connect(function(hit)
                    if hit:IsA("Tool") then
                        player:TakeDamage(10) -- Dano ao jogador
                    end
                end)
            end
        end
    end
end

-- Função para ativar/desativar Fly
local function toggleFly()
    flyEnabled = not flyEnabled
    if flyEnabled then
        local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
        local bodyGyro = Instance.new("BodyGyro", humanoidRootPart)
        local bodyVelocity = Instance.new("BodyVelocity", humanoidRootPart)
        bodyGyro.MaxTorque = Vector3.new(4000, 4000, 4000)
        bodyGyro.P = 3000
        bodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)

        UserInputService.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Touch then
                -- Para tocar na tela e ativar o movimento de voo
                bodyVelocity.Velocity = Vector3.new(0, 50, 0) -- Voo para cima
            end
        end)

        -- Desativar Fly quando o jogador não estiver mais voando
        bodyGyro:Destroy()
        bodyVelocity:Destroy()
    end
end

-- Função para ativar/desativar Auto CL
local function toggleAutoCL()
    autoCLEnabled = not autoCLEnabled
    if autoCLEnabled then
        LocalPlayer.Character.Humanoid.Died:Connect(function()
            wait(1) -- Esperar um segundo antes de quitar
            LocalPlayer:Kick("Você foi removido por morrer.")
        end)
    end
end

-- Anti Staff: Remove membros da equipe staff
Players.PlayerAdded:Connect(function(player)
    if player:GetRankInGroup(YOUR_GROUP_ID) > 0 then -- Substitua YOUR_GROUP_ID pelo ID do seu grupo
        player:Kick("Você foi removido por ser um membro da equipe staff.")
    end
end)

-- Auto Revistar: Receber todas as armas do jogador
UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
    if not gameProcessedEvent and input.KeyCode == Enum.KeyCode.R then -- Use R para ativar o revistar
        if autoRevistarEnabled then
            for _, tool in ipairs(LocalPlayer.Backpack:GetChildren()) do
                if tool:IsA("Tool") then
                    tool.Parent = LocalPlayer.Character -- Mover as armas para o personagem
                end
            end
        end
    end
end)

-- Criar botões de ativação e desativação no Menu
local function createToggleButton(name, position, toggleFunction)
    local toggleButton = Instance.new("TextButton", MenuFrame)
    toggleButton.Size = UDim2.new(0, 100, 0, 50)
    toggleButton.Position = UDim2.new(0.5, -50, position)
    toggleButton.BackgroundColor3 = Color3.fromRGB(255, 255, 0) -- Amarelo para indicar que é um botão
    toggleButton.TextColor3 = Color3.fromRGB(0, 0, 0)
    toggleButton.Text = name

    toggleButton.MouseButton1Click:Connect(function()
        toggleFunction()
        toggleButton.BackgroundColor3 = toggleButton.BackgroundColor3 == Color3.fromRGB(255, 255, 0) and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 255, 0) -- Alterna a cor
    end)

    toggleButton.TouchTap:Connect(function() -- Para dispositivos móveis
        toggleFunction()
        toggleButton.BackgroundColor3 = toggleButton.BackgroundColor3 == Color3.fromRGB(255, 255, 0) and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 255, 0) -- Alterna a cor
    end)
end

createToggleButton("Auto Revistar", 0.1, toggleAutoRevistar)
createToggleButton("Hitbox", 0.2, toggleHitbox)
createToggleButton("Fly", 0.3, toggleFly)
createToggleButton("Auto CL", 0.4, toggleAutoCL)

-- Funções para minimizar e maximizar o menu
MinimizeButton.TouchTap:Connect(function() -- Para dispositivos móveis
    MenuFrame.Visible = false
    MaximizeButton.Visible = true
end)

MaximizeButton.TouchTap:Connect(function() -- Para dispositivos móveis
    MenuFrame.Visible = true
    MaximizeButton.Visible = false
end)

-- Boost de FPS (Exemplo simples)
RunService.RenderStepped:Connect(function()
    game:GetService("UserSettings").GameSettings.MasterVolume = 0 -- Desativa o volume do jogo
end)

print("✅ Menu Quadrado Azul (Compatível com Celular) com funções criadas com sucesso!")
