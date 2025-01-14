loadstring([[
-- Criação da GUI
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")  -- O quadrado
local TitleLabel = Instance.new("TextLabel")  -- O título
local RollbackButton = Instance.new("TextButton")
local StopButton = Instance.new("TextButton")
local RejoinButton = Instance.new("TextButton")

-- Configurações gerais do ScreenGui
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.Name = "RollbackGUI"

-- Configurações do Frame (quadrado)
MainFrame.Parent = ScreenGui
MainFrame.Size = UDim2.new(0, 300, 0, 250)  -- Ajuste o tamanho para que caibam os botões
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -125)  -- Ajuste para centralizar melhor
MainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)  -- Cor preta
MainFrame.BackgroundTransparency = 0.5  -- Transparência no fundo
MainFrame.BorderSizePixel = 3
MainFrame.BorderColor3 = Color3.fromRGB(255, 255, 255)  -- Borda branca

-- Configurações do Título
TitleLabel.Parent = MainFrame
TitleLabel.Size = UDim2.new(1, 0, 0.2, 0)  -- Ajuste o tamanho do título
TitleLabel.Position = UDim2.new(0, 0, 0, 0)
TitleLabel.Text = "Guibbs Rollback - Anime Adventures"
TitleLabel.TextSize = 12  -- Fonte menor
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)  -- Cor branca
TitleLabel.BackgroundTransparency = 1  -- Sem fundo
TitleLabel.TextAlign = Enum.TextXAlignment.Center

-- Configurações do botão "Turn On Rollback"
RollbackButton.Parent = MainFrame
RollbackButton.Size = UDim2.new(0, 200, 0, 50)
RollbackButton.Position = UDim2.new(0.5, -100, 0.3, 0)
RollbackButton.Text = "Turn On Rollback"
RollbackButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)  -- Cor do botão branca
RollbackButton.TextColor3 = Color3.fromRGB(0, 0, 0)  -- Cor do texto preta

-- Configurações do botão "Turn Off Rollback"
StopButton.Parent = MainFrame
StopButton.Size = UDim2.new(0, 200, 0, 50)
StopButton.Position = UDim2.new(0.5, -100, 0.5, 0)
StopButton.Text = "Turn Off Rollback"
StopButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)  -- Cor do botão branca
StopButton.TextColor3 = Color3.fromRGB(0, 0, 0)  -- Cor do texto preta

-- Configurações do botão "Reentrar"
RejoinButton.Parent = MainFrame
RejoinButton.Size = UDim2.new(0, 200, 0, 50)
RejoinButton.Position = UDim2.new(0.5, -100, 0.7, 0)
RejoinButton.Text = "Reentrar no Jogo"
RejoinButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)  -- Cor do botão branca
RejoinButton.TextColor3 = Color3.fromRGB(0, 0, 0)  -- Cor do texto preta

local rollbackActive = false
local originalValues = {}

-- Função para ativar o rollback
RollbackButton.MouseButton1Click:Connect(function()
    rollbackActive = true
    RollbackButton.Text = "Rollback Ativado"
    print("Rollback ativado!")

    -- Salva os valores atuais como "estado inicial"
    if game.Players.LocalPlayer:FindFirstChild("leaderstats") then
        for _, v in pairs(game.Players.LocalPlayer.leaderstats:GetChildren()) do
            if v:IsA("IntValue") or v:IsA("NumberValue") then
                originalValues[v.Name] = v.Value
                print("Valor salvo: " .. v.Name .. " = " .. v.Value)  -- Log de Debug
            end
        end
    else
        print("leaderstats não encontrado!")  -- Log de Debug
    end
end)

-- Função para desativar o rollback
StopButton.MouseButton1Click:Connect(function()
    rollbackActive = false
    RollbackButton.Text = "Turn On Rollback"
    print("Rollback desativado!")
end)

-- Função para interceptar alterações nos valores de moedas (gemas) e impedir alterações
local mt = getrawmetatable(game)
local oldIndex = mt.__newindex

setreadonly(mt, false)
mt.__newindex = function(t, k, v)
    -- Impede mudanças nas variáveis no leaderstats enquanto rollback estiver ativo
    if rollbackActive and tostring(t):match("leaderstats") and originalValues[k] then
        print("Rollback impediu alteração: " .. tostring(k) .. " -> " .. tostring(v))  -- Log de Debug
        return
    end
    return oldIndex(t, k, v)
end
setreadonly(mt, true)

-- Função para reentrar no jogo
RejoinButton.MouseButton1Click:Connect(function()
    local TeleportService = game:GetService("TeleportService")
    local player = game.Players.LocalPlayer
    print("Reentrando no jogo...")  -- Log de Debug
    TeleportService:Teleport(game.PlaceId, player)
end)

-- Função para restaurar os dados ao reentrar
game.Players.LocalPlayer.Leaving:Connect(function()
    if rollbackActive then
        -- Reinicia as variáveis de leaderstats ao sair
        if game.Players.LocalPlayer:FindFirstChild("leaderstats") then
            for _, v in pairs(game.Players.LocalPlayer.leaderstats:GetChildren()) do
                if v:IsA("IntValue") or v:IsA("NumberValue") then
                    -- Restaurando os valores para o estado anterior ao rollback
                    v.Value = originalValues[v.Name] or v.Value
                    print("Valor restaurado após rollback: " .. v.Name .. " = " .. v.Value)
                end
            end
        end
    end
end)

print("Script de rollback com GUI e rejoin carregado com sucesso!")
]])()
