
import org.testng.annotations.Test;
import java.text.DateFormat;
import java.text.Format;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
import org.openqa.selenium.Alert;
import org.openqa.selenium.By;
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;

public class New {
	String url = "http://damp-sands-8561.herokuapp.com";
	String successSignInMsg = "Signed in successfully.";
	String successEntryMsg = "Entry was successfully created.";
	String errorMessage = "Maximum entries reached for this date.";
	String infoMessage = "Only 4 entries allowed per day";
	String entryPage = "/entries";
	String reportPage = "/report";
	int bloodGlucoseReadings[] = { 7, 8, 9, 6, 4 };
	WebDriver driver = new FirefoxDriver();
	WebDriverWait wait = new WebDriverWait(driver, 10);
	String currentDate = "";
	String currentMonth = "";
	int totalReadings = 0;
	int totalReadingsForMoth = 0;
	String userName = "raghu.yel002@gmail.com";
	String passWord = "codetheoryio";
	static String email = "user_email";
	static String password = "user_password";
	static String submit = "commit";
	static String successMsg = "//div[contains(text(),'Signed in successfully.')]";
	static String deleteEntries = "//a[contains(.,'Delete')]";
	static String entrySuccessMsg = "//div[contains(text(),'Entry was successfully created')]";
	static String todaysReadings = "//h3[contains(.,'Daily Report as of')]";
	static String selectingMonthlyReadings = "//a[contains(.,'Monthly')]";
	static String verifyMonth = "//h3[contains(.,'Monthly Report as of')]";
	static String monthlyReadings = "//h3[contains(.,'Monthly Report as of')]";
	static String totalBloodReadings = "//table[@class='table table-condensed']//tbody//td[3]";
	static String highestBloodReading = "//table[@class='table table-condensed']//tbody//td[5]";
	static String lowestBloodReading = "//table[@class='table table-condensed']//tbody//td[4]";
	static String averageBloodReading = "//table[@class='table table-condensed']//tbody//td[6]";
	static String errorMsg = "//div[@class='alert alert-warning fade in']";
	static String infoMsg = "//div[@class='alert alert-info']";
	static String addEntryButton = "//a[contains(text(),'Add new')]";
	static String entryTextBox = "entry_level";

	@BeforeMethod(alwaysRun = true)
	public void loginUser() {
		driver.get(url);
		driver.manage().window().maximize();
		driver.findElement(By.id(email)).sendKeys(userName);
		driver.findElement(By.id(password)).sendKeys(passWord);
		driver.findElement(By.name(submit)).submit();
		// Verifying user is logged in successfully.
		Assert.assertTrue(driver.findElement(By.xpath(successMsg)).getText()
				.equalsIgnoreCase(successSignInMsg),
				"User is not registerd or entering wrong UserName or password");
	}

	public void getCurrentDate() {
		DateFormat dateFormat = new SimpleDateFormat("yyyy/MM/dd");
		Date date = new Date();
		currentDate = dateFormat.format(date);
	}

	@AfterMethod(alwaysRun = true)
	public void deleteBloodReadings() {
		navigateToPage(entryPage);
		try {
			while (driver.findElement(By.xpath(deleteEntries)).isEnabled()) {
				driver.findElement(By.xpath(deleteEntries)).click();
				Alert alt = driver.switchTo().alert();
				alt.accept();
			}
		} catch (NoSuchElementException e) {
			System.out.println(e);
		}
		driver.manage().deleteAllCookies();
	}

	@Test(description = "Enter the blood values and verify the report")
	public void enterBloodEntryAndVerifyReport() throws InterruptedException {
		// Navigate to Level Entries.
		navigateToPage(entryPage);
		// Entering blood values.
		for (int i = 0; i < 4; i++) {
			addEntries(bloodGlucoseReadings[i]);
		}
		// Verifying success message after entering blood values.
		Assert.assertTrue(driver.findElement(By.xpath(entrySuccessMsg))
				.getText().equalsIgnoreCase(successEntryMsg),
				"Not able to enter valid blood entries");
		// Navigate to reports.
		navigateToPage(reportPage);
		// Verify by default it is showing todays readings.
		Assert.assertTrue(driver.findElement(By.xpath(todaysReadings))
				.getText().contains(currentDate),
				"Report not displaying todays readings");
		wait.until(ExpectedConditions.visibilityOfElementLocated(By
				.xpath(totalBloodReadings)));
		// Verify the readings of the day are same as entered
		List<WebElement> readings = driver.findElements(By
				.xpath(totalBloodReadings));
		for (int i = 0; i < readings.size(); i++) {
			// Verifying all the values are same as we entered
			Assert.assertEquals(Integer.parseInt(readings.get(i).getText()),
					bloodGlucoseReadings[i], "Following value " + i
							+ "is not same as patient entered");
			int highestReading = Integer.parseInt(driver
					.findElement(By.xpath(highestBloodReading)).getText()
					.trim());
			int lowestReading = Integer
					.parseInt(driver.findElement(By.xpath(lowestBloodReading))
							.getText().trim());
			int allReading = Integer.parseInt(readings.get(i).getText());
			if (lowestReading > allReading) {
				// Verifying Lowest reading
				Assert.assertTrue(false,
						"Lowest reading is not matching with entered value");
			}
			if (highestReading < allReading) {
				// Verifying Highest reading
				Assert.assertTrue(false,
						"Highest reading is not matching with entered value");
			}
			totalReadings = totalReadings + allReading;
		}
		int actualAverageReading = Integer.parseInt(driver.findElement(
				By.xpath(averageBloodReading)).getText());
		int ExpectedAverageReading = Math
				.round(totalReadings / readings.size());
		// Verifying average value.
		Assert.assertEquals(actualAverageReading, ExpectedAverageReading,
				"Average value for day is not mathching");
		// Select monthly report and verify the same
		driver.findElement(By.xpath(selectingMonthlyReadings)).click();
		wait.until(ExpectedConditions.visibilityOfElementLocated(By
				.xpath(verifyMonth)));
		currentMonth = getcurrentmonth();
		// Verify monthly readings.
		Assert.assertTrue(driver.findElement(By.xpath(verifyMonth)).getText()
				.contains(currentMonth),
				"Report not displaying Monthly readings");
		// verify readings for the month
		readings = driver.findElements(By.xpath(totalBloodReadings));
		for (int i = 0; i < readings.size(); i++) {
			// Verifying all the values are same as we entered
			Assert.assertEquals(Integer.parseInt(readings.get(i).getText()),
					bloodGlucoseReadings[i], "Following value " + i
							+ "for month is not same as patient entered");
			int highestReading = Integer.parseInt(driver
					.findElement(By.xpath(highestBloodReading)).getText()
					.trim());
			int lowestReading = Integer
					.parseInt(driver.findElement(By.xpath(lowestBloodReading))
							.getText().trim());
			int allReading = Integer.parseInt(readings.get(i).getText());
			if (lowestReading > allReading) {
				// Verifying Lowest reading
				Assert.assertTrue(false,
						"Lowest reading for month is not matching with entered value");
			}
			if (highestReading < allReading) {
				// Verifying Highest reading
				Assert.assertTrue(false,
						"Highest reading for month is not matching with entered value");
			}
			totalReadingsForMoth = totalReadingsForMoth + allReading;
		}
		actualAverageReading = Integer.parseInt(driver.findElement(
				By.xpath(averageBloodReading)).getText());
		ExpectedAverageReading = Math.round(totalReadingsForMoth
				/ readings.size());
		// Verifying average value.
		Assert.assertEquals(actualAverageReading, ExpectedAverageReading,
				"Average value for month is not mathching");

	}

	@Test(description = "Enter the blood redings more than four times and verify the error")
	public void enterBloodValuesMoreThanFourTimesInADay() {
		// Navigate to Level Entries.
		navigateToPage(entryPage);
		// Entering blood values.
		for (int i = 0; i < 5; i++) {
			addEntries(bloodGlucoseReadings[i]);
		}
		// Verify error message saying cannot enter more than 4 times a day
		Assert.assertTrue(driver.findElement(By.xpath(errorMsg)).getText()
				.contains(errorMessage),
				"Error message is not displaying after  4th entry");
		// verify info message
		Assert.assertTrue(driver.findElement(By.xpath(infoMsg)).getText()
				.trim().contains(infoMessage),
				"Info message is not displaying after  4th entry");
	}

	public void addEntries(int bloodGlucoseReadings) {
		driver.findElement(By.xpath(addEntryButton)).click();
		wait.until(ExpectedConditions.visibilityOfElementLocated(By
				.id(entryTextBox)));
		driver.findElement(By.id(entryTextBox)).sendKeys(
				"" + bloodGlucoseReadings);
		driver.findElement(By.name(submit)).submit();
	}

	public String getcurrentmonth() {
		Format formatter = new SimpleDateFormat("MMMM");
		currentMonth = formatter.format(new Date());
		return currentMonth;
	}

	public void navigateToPage(String page) {
		System.out.println(url + page);
		driver.get(url + page);
	}
}
