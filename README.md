import com.google.gson.Gson;
import com.google.gson.JsonObject;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;
import javax.swing.JOptionPane;

public class Main {
    private static String sourceCurrency;
    private static String targetCurrency;
    private static final List<String> conversionHistory = new ArrayList<>();

    public static void main(String[] args) {
        while (true) {
            String[] options = {"Convertir moneda",
                    "Consultar historial",
                    "Salir"};
            int option = JOptionPane.showOptionDialog(null, "Seleccione una opción:", "Menú principal",
                    JOptionPane.DEFAULT_OPTION, JOptionPane.PLAIN_MESSAGE,
                    null, options, options[0]);

            switch (option) {
                case 0:
                    convertCurrency();
                    break;
                case 1:
                    viewConversionHistory();
                    break;
                case 2:
                    System.out.println("Gracias por utilizar el conversr de monedas");
                    System.exit(0);
                default:
                    System.out.println("Opción inválida. Por favor, seleccione una opción válida.");
            }
        }
    }

    private static void convertCurrency() {
        String[] currencies = {"Peso Mexicano (MXN)", "Peso Argentino (ARS)", "Peso Colombiano (COP)", "Dólar (USD)", "Euro (EUR)", "Libra Esterlina (GBP)", "Yen Japonés (JPY)", "Won Sul-Coreano (KRW)"};

        // Selección de moneda de origen
        if (sourceCurrency == null || targetCurrency == null) {
            String selectedSourceCurrency = (String) JOptionPane.showInputDialog(null, "Selecciona la moneda de origen:",
                    "Convertir moneda",
                    JOptionPane.PLAIN_MESSAGE, null, currencies, currencies[0]);
            sourceCurrency = getCurrencyCode(selectedSourceCurrency);

            // Selección de moneda de destino
            String selectedTargetCurrency = (String) JOptionPane.showInputDialog(null, "Selecciona la moneda de destino:",
                    "Convertir moneda",
                    JOptionPane.PLAIN_MESSAGE, null, currencies, currencies[0]);
            targetCurrency = getCurrencyCode(selectedTargetCurrency);
        }

        // Ingreso de cantidad a convertir
        String amountStr = JOptionPane.showInputDialog("Ingrese la cantidad a convertir:");
        if (amountStr != null && !amountStr.isEmpty()) {
            try {
                double amount = Double.parseDouble(amountStr);
                double conversionRate = getConversionRate(sourceCurrency, targetCurrency);
                double result = amount * conversionRate;
                String message = "Resultado de la conversión: " + amount + " " + sourceCurrency + " = " + result + " " + targetCurrency;
                JOptionPane.showMessageDialog(null, message, "Resultado", JOptionPane.INFORMATION_MESSAGE);
                conversionHistory.add("Convertido " + amount + " " + sourceCurrency + " a " + result + " " + targetCurrency);

                // Preguntar si desea convertir otra cantidad
                int choice = JOptionPane.showConfirmDialog(null, "¿Desea convertir otra cantidad de " + sourceCurrency + " a " + targetCurrency + "?", "Convertir otra cantidad", JOptionPane.YES_NO_OPTION);
                if (choice == JOptionPane.YES_OPTION) {
                    convertCurrency();
                } else {
                    // Reiniciar las monedas seleccionadas
                    sourceCurrency = null;
                    targetCurrency = null;
                }

            } catch (NumberFormatException e) {
                JOptionPane.showMessageDialog(null, "Por favor ingrese un número válido.", "Error", JOptionPane.ERROR_MESSAGE);
            } catch (Exception e) {
                JOptionPane.showMessageDialog(null, "Error al realizar la conversión: " + e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    private static void viewConversionHistory() {
        StringBuilder historyMessage = new StringBuilder("----- HISTORIAL DE CONVERSIONES -----\n");
        if (conversionHistory.isEmpty()) {
            historyMessage.append("El historial está vacío.");
        } else {
            for (String entry : conversionHistory) {
                historyMessage.append(entry).append("\n");
            }
        }
        historyMessage.append("--------------------------------------");
        JOptionPane.showMessageDialog(null, historyMessage.toString(), "Historial de Conversiones", JOptionPane.PLAIN_MESSAGE);
    }

    private static String getCurrencyCode(String currencyName) {
        switch (currencyName) {
            case "Peso Mexicano (MXN)":
                return "MXN";
            case "Peso Argentino (ARS)":
                return "ARS";
            case "Peso Colombiano (COP)":
                return "COP";
            case "Dólar (USD)":
                return "USD";
            case "Euro (EUR)":
                return "EUR";
            case "Libra Esterlina (GBP)":
                return "GBP";
            case "Yen Japonés (JPY)":
                return "JPY";
            case "Won Sul-Coreano (KRW)":
                return "KRW";
            default:
                return null;
        }
    }

    private static double getConversionRate(String baseCurrency, String targetCurrency) throws Exception {
        String url = "https://v6.exchangerate-api.com/v6/1037ae973d901a6844f1f32b/latest/" + baseCurrency;
        URL apiUrl = new URL(url);
        HttpURLConnection connection = (HttpURLConnection) apiUrl.openConnection();
        connection.setRequestMethod("GET");

        int responseCode = connection.getResponseCode();
        if (responseCode == HttpURLConnection.HTTP_OK) {
            BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            StringBuilder response = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                response.append(line);
            }
            reader.close();

            Gson gson = new Gson();
            JsonObject jsonObject = gson.fromJson(response.toString(), JsonObject.class);

            JsonObject conversionRates = jsonObject.getAsJsonObject("conversion_rates");
            if (conversionRates != null && conversionRates.has(targetCurrency)) {
                return conversionRates.get(targetCurrency).getAsDouble();
            } else {
                throw new Exception("La tasa de conversión para " + targetCurrency + " no está disponible en el JSON recibido");
            }
        } else {
            throw new Exception("Error al obtener la tasa de conversión. Código de respuesta: " + responseCode);
        }
    }
}
